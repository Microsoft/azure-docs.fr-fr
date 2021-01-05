---
title: Problèmes lors de la configuration de l’approvisionnement des utilisateurs pour une application relevant de la galerie Azure AD
description: Comment résoudre les problèmes courants rencontrés lors de la configuration l’approvisionnement des utilisateurs pour une application déjà répertoriée dans la galerie d’applications Azure AD
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: troubleshooting
ms.date: 09/03/2019
ms.author: kenwith
ms.reviewer: asteen
ms.openlocfilehash: 306c3771c0392bbc97260a726e153cfd385cebcd
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95994789"
---
# <a name="problem-configuring-user-provisioning-to-an-azure-ad-gallery-application"></a>Problèmes lors de la configuration de l’approvisionnement des utilisateurs pour une application relevant de la galerie Azure AD

La configuration de [l’approvisionnement automatique des utilisateurs](user-provisioning.md) pour une application (lorsqu’il est pris en charge) implique de respecter des instructions spécifiques pour la préparation de l’application. Vous pouvez ensuite utiliser le Portail Azure afin de configurer le service d’approvisionnement pour synchroniser les comptes d’utilisateur avec l’application.

Vous devez toujours commencer par rechercher le didacticiel spécifique à la configuration de l’approvisionnement pour votre application. Suivez ensuite ces étapes afin de configurer l’application et Azure AD pour créer la connexion d’approvisionnement. Vous trouverez une liste de didacticiels pour les applications à la page [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](../saas-apps/tutorial-list.md).

## <a name="how-to-see-if-provisioning-is-working"></a>Comment savoir si l’approvisionnement fonctionne 

Une fois que le service est configuré, la plupart des informations sur son fonctionnement peuvent être obtenues à deux emplacements :

-   **Journaux de provisionnement (préversion)**  : les [journaux de provisionnement](../reports-monitoring/concept-provisioning-logs.md?context=azure/active-directory/manage-apps/context/manage-apps-context) enregistrent toutes les opérations effectuées par le service de provisionnement, y compris l’interrogation d’Azure AD concernant les utilisateurs attribués qui se trouvent dans l’étendue du provisionnement. Interrogez l’application cible concernant l’existence de ces utilisateurs en comparant les objets utilisateur du système. Ajoutez ensuite, mettez à jour ou désactivez le compte d’utilisateur dans le système cible en fonction de cette comparaison. Vous pouvez accéder aux journaux d’approvisionnement dans le portail Azure en sélectionnant **Azure Active Directory** &gt; **Applications d’entreprise** &gt; **Journaux de provisionnement (préversion)** dans la section **Activité**.

-   **État actuel :** un résumé du dernier approvisionnement pour une application donnée est visible sous **Azure Active Directory &gt; Applications d’entreprise &gt; \[Nom de l’application\] &gt;Approvisionnement**, en bas de l’écran, sous les paramètres du service. La section État actuel indique si un cycle a démarré le provisionnement de comptes d’utilisateurs. Vous pouvez surveiller la progression du cycle, voir combien d’utilisateurs et de groupes ont été approvisionnés et combien de rôles sont créés. En cas d’erreur, les détails se trouvent dans les journaux d’approvisionnement (../reports-monitoring/concept-provisioning-logs.md?context=azure/active-directory/manage-apps/context/manage-apps-context).

## <a name="general-problem-areas-with-provisioning-to-consider"></a>Problèmes d’approvisionnement généraux à prendre en compte

Voici une liste des problèmes généraux que vous pouvez explorer si vous savez par où commencer.

* [Le service d’approvisionnement ne semble pas démarrer](#provisioning-service-does-not-appear-to-start)
* Impossible d’enregistrer la configuration car les informations d’identification de l’application ne fonctionnent pas
* [Les journaux de provisionnement indiquent que les utilisateurs sont « ignorés » et non provisionnés, bien qu’ils soient attribués](#provisioning-logs-say-users-are-skipped-and-not-provisioned-even-though-they-are-assigned)

## <a name="provisioning-service-does-not-appear-to-start"></a>Le service d’approvisionnement ne semble pas démarrer

Si vous définissez le paramètre **État de l’approvisionnement** sur **Activé** dans la section **Azure Active Directory &gt; Applications d’entreprise &gt; \[Nom de l’application\] &gt;Approvisionnement** du portail Azure. Cependant aucune autre information d’état n’apparaît sur cette page après de nouveaux chargements. Le service est probablement en cours d’exécution mais n’a pas encore terminé le cycle initial. Vérifiez les **Journaux de provisionnement** décrits ci-dessus pour déterminer les opérations effectuées par le service, et la présence éventuelle d’erreurs.

>[!NOTE]
>Un cycle initial peut prendre de 20 minutes à plusieurs heures, en fonction de la taille de l’annuaire Azure AD et du nombre d’utilisateurs présents dans l’étendue du provisionnement. Les synchronisations qui suivent le cycle initial sont plus rapides, car le service de provisionnement stocke les filigranes qui représentent l’état des deux systèmes après le cycle initial, ce qui améliore les performances des synchronisations suivantes.
>
>

## <a name="cant-save-configuration-due-to-app-credentials-not-working"></a>Impossible d’enregistrer la configuration car les informations d’identification de l’application ne fonctionnent pas

Pour un fonctionnement correct de l’approvisionnement, Azure AD nécessite des informations d’identification valides qui lui permettent de se connecter à une API de gestion des utilisateurs fournie par cette application. Si ces informations d’identification ne fonctionnent pas ou si vous ne les connaissez pas, consultez le didacticiel dédié à la configuration de cette application (décrit précédemment).

## <a name="provisioning-logs-say-users-are-skipped-and-not-provisioned-even-though-they-are-assigned"></a>Les journaux de provisionnement indiquent que les utilisateurs sont ignorés et non provisionnés, bien qu’ils soient attribués

Lorsqu’un utilisateur apparaît comme « ignoré » dans les journaux de provisionnement, il est capital de consulter les détails contenus dans le message du journal pour en déterminer la raison. Voici les raisons les plus courantes et les solutions correspondantes :

- **Un filtre d’étendue a été configuré,** **qui exclut l’utilisateur en fonction d’une valeur d’attribut**. Pour plus d’informations, voir [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

- **L’utilisateur n’est pas « autorisé de manière effective ».** Ce message d’erreur indique un problème concernant l’enregistrement d’affectation d’utilisateurs stocké dans Azure AD. Pour résoudre ce problème, supprimez l’affectation de l’utilisateur (ou du groupe) à partir de l’application, puis réaffectez l’utilisateur. Pour plus d’informations, voir [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md).

- **Un attribut requis manque ou n’est pas indiqué pour un utilisateur.** Lors de la configuration de l’approvisionnement, il est important de vérifier et configurer les mappages d’attributs et les workflows qui définissent les propriétés de l’utilisateur (ou du groupe) passant d’Azure AD à l’application. Cela inclut la définition d’une « propriété correspondante » réservée à l’utilisation et à la mise en correspondance des utilisateurs/groupes entre les deux systèmes. Pour plus d’informations sur ce processus important, voir [Personnalisation des mappages d’attributs d’approvisionnement d’utilisateurs](../app-provisioning/customize-application-attributes.md).

  * **Mappage d’attributs pour les groupes :** Approvisionnement du nom du groupe et des détails du groupe, en plus des membres, si la prise en charge est effective pour certaines applications. Vous pouvez activer ou désactiver cette fonctionnalité en activant ou désactivant le **mappage** pour les objets de groupe affichés dans l’onglet **Approvisionnement**. Si les groupes d’approvisionnement sont activés, veillez à passer en revue les mappages d’attributs afin de vous assurer qu’un champ approprié est utilisé pour l’« ID correspondant ». Il peut s’agir du nom d’affichage ou de l’alias de courrier électronique dans la mesure où le groupe et ses membres ne sont pas approvisionnés si la propriété correspondante est vide ou n’est pas remplie pour un groupe dans Azure AD.

## <a name="next-steps"></a>Étapes suivantes
[Automatisation de l’approvisionnement et de la l’annulation de l’approvisionnement des utilisateurs pour les applications SaaS avec Azure Active Directory](user-provisioning.md)
