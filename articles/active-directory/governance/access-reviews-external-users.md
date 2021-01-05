---
title: Utiliser Azure AD Identity Governance pour examiner et supprimer les utilisateurs externes qui n’ont plus accès aux ressources
description: Utiliser les révisions d’accès pour étendre ou supprimer l’accès des membres des organisations partenaires
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.subservice: compliance
ms.date: 09/06/2020
ms.author: barclayn
ms.openlocfilehash: 19f88da6a678221cde66bf61668d16ba9ab998a4
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92677313"
---
# <a name="use-azure-active-directory-azure-ad-identity-governance-to-review-and-remove-external-users-who-no-longer-have-resource-access"></a>Utiliser Azure Active Directory (Azure AD) Identity Governance pour examiner et supprimer les utilisateurs externes qui n’ont plus accès aux ressources

Cet article décrit les fonctionnalités et les méthodes qui vous permettent d’identifier et de sélectionner des identités externes afin de les examiner et de les supprimer d’Azure AD si elles ne sont plus nécessaires. Le cloud facilite plus que jamais la collaboration avec des utilisateurs internes ou externes. En adoptant Office 365, les organisations commencent à constater la prolifération des identités externes (notamment les invités), car les utilisateurs collaborent sur des données, des documents ou des espaces de travail numériques comme Teams. Les organisations doivent trouver un équilibre, en permettant la collaboration et en répondant aux exigences en matière de sécurité et de gouvernance. Une partie de ces efforts doit inclure l’évaluation et le nettoyage des utilisateurs externes, qui ont été invités à collaborer dans votre locataire et qui proviennent d’organisations partenaires, et leur suppression de votre service Azure AD lorsqu’ils ne sont plus nécessaires.

>[!NOTE]
>L’utilisation des révisions d’accès Azure AD requiert une licence payante ou d’essai Azure AD Premium P2 ou Enterprise Mobility + Security E5 valide. Pour plus d’informations, consultez la page [Éditions d’Azure Active Directory](../fundamentals/active-directory-whatis.md).

## <a name="why-review-users-from-external-organizations-in-your-tenant"></a>Pourquoi examiner les utilisateurs d’organisations externes dans votre locataire ?

Dans la plupart des organisations, les utilisateurs finaux invitent des partenaires commerciaux et des fournisseurs en vue d’une collaboration. La nécessité de collaborer pousse les organisations à fournir aux propriétaires de ressources et aux utilisateurs finaux un moyen d’évaluer et d’attester régulièrement les utilisateurs externes. Souvent, le processus d’intégration de nouveaux partenaires de collaboration est planifié et comptabilisé, mais comme de nombreuses collaborations n’ont pas de date de fin précise, il n’est pas toujours évident de savoir quand un utilisateur n’a plus besoin de l’accès. De plus, la gestion du cycle de vie des identités incite les entreprises à nettoyer régulièrement Azure AD et à supprimer les utilisateurs qui n’ont plus besoin d’accéder aux ressources de l’organisation. Le fait de ne conserver dans le répertoire que les références d’identité pertinentes des partenaires et des fournisseurs permet de réduire le risque que vos employés sélectionnent et accordent par inadvertance l’accès à des utilisateurs externes qui auraient dû être supprimés. Ce document vous guide à travers plusieurs options allant des suggestions proactives recommandées aux activités de réactivité et de nettoyage pour régir les identités externes.

## <a name="use-entitlement-management-to-grant-and-revoke-access"></a>Utiliser la gestion des droits d’utilisation pour accorder et révoquer l’accès

Les fonctionnalités de la gestion des droits d’utilisation permettent au [cycle de vie automatisé des identités externes](entitlement-management-external-users.md#manage-the-lifecycle-of-external-users) d’accéder aux ressources. En établissant des processus et des procédures pour gérer l’accès via la gestion des droits d’utilisation et en publiant des ressources via des packages d’accès, le suivi de l’accès des utilisateurs externes aux ressources devient un problème beaucoup moins compliqué à résoudre. Lorsque vous gérez l’accès via des [packages d’accès de gestion des droits d’utilisation](entitlement-management-overview.md) dans Azure AD, votre organisation peut définir et gérer de manière centralisée l’accès de vos utilisateurs, ainsi que celui des utilisateurs des organisations partenaires. La gestion des droits d’utilisation utilise des approbations et des attributions de packages d’accès pour effectuer le suivi des demandes d’accès des utilisateurs externes. Si un utilisateur externe perd toutes ses attributions, la gestion des droits d’utilisation peut supprimer automatiquement ces utilisateurs externes du locataire. 

## <a name="find-guests-not-invited-through-entitlement-management"></a>Rechercher des invités non autorisés via la gestion des droits d’utilisation

Lorsque les employés sont autorisés à collaborer avec des utilisateurs externes, ils peuvent inviter un nombre quelconque d’utilisateurs extérieurs à votre organisation. Il n’est pas toujours possible de rechercher et de regrouper des partenaires externes dans des groupes dynamiques alignés sur l’entreprise et de les examiner, car il peut y avoir trop d’entreprises différentes à examiner, ou il n’y a pas de propriétaire ni de sponsor pour l’organisation. Microsoft fournit un exemple de script PowerShell qui peut vous aider à analyser l’utilisation des identités externes dans un locataire. Le script énumère les identités externes et les catégorise. Le script peut vous aider à identifier et à nettoyer les identités externes qui ne sont peut-être plus nécessaires. Dans le cadre de la sortie du script, l’exemple de script prend en charge la création automatisée de groupes de sécurité qui contiennent les partenaires externes sans groupe identifié, à des fins d’analyse et d’utilisation plus poussées avec les révisions d’accès Azure AD.
Le script est accessible sur le site de [GitHub](https://github.com/microsoft/access-reviews-samples/tree/master/ExternalIdentityUse). Une fois l’exécution du script terminée, il génère un fichier de sortie HTML qui présente les identités externes qui :

- n’appartiennent plus à un groupe du locataire ;
- ont un rôle privilégié attribué dans le locataire ;
- ont une application attribuée dans le locataire.

La sortie comprend également les différents domaines de chacune de ces identités externes. 

>[!NOTE]
>Le script référencé ci-dessus est un exemple de script qui vérifie l’appartenance à un groupe, les attributions de rôles et les attributions d’applications dans Azure AD. Il peut y avoir d’autres attributions dans les applications que les utilisateurs externes ont reçues en dehors d’Azure AD, telles que SharePoint (attribution directe d’appartenance), Azure RBAC ou Azure DevOps.

## <a name="review-resources-used-by-external-identities"></a>Examiner les ressources utilisées par les identités externes

Si vous avez des identités externes utilisant des ressources telles que Teams ou d’autres applications qui ne sont pas encore régies par la gestion des droits d’utilisation, nous vous conseillons également de réviser régulièrement l’accès à ces ressources. Les [révisions d’accès](create-access-review.md) Azure AD vous donne la possibilité de réviser l’accès des identités externes en laissant le propriétaire de la ressource, les identités externes elles-mêmes ou une autre personne déléguée à laquelle vous faites confiance attester que l’accès continu est nécessaire. Les révisions d’accès ciblent une ressource et créent une activité de vérification qui s’adresse soit à toute personne ayant accès à la ressource, soit aux utilisateurs invités uniquement. Le réviseur verra alors la liste des utilisateurs qu’il doit vérifier : tous les utilisateurs, y compris les employés de votre organisation, ou uniquement les identités externes.

![utilisation d’un groupe pour réviser l’accès](media/access-reviews-external-users/group-members.png)

L’établissement d’une culture de révision axée sur le propriétaire de la ressource contribue à régir l’accès des identités externes. Dans la plupart des cas, les propriétaires de ressources, qui sont responsables de l’accès, de la disponibilité et de la sécurité des informations qu’ils possèdent, sont votre meilleur public pour prendre des décisions concernant l’accès à leurs ressources et sont plus proches des utilisateurs qui y accèdent que l’équipe informatique centrale ou un sponsor qui gère de nombreux externes.

## <a name="create-access-reviews-for-external-identities"></a>Créer des révisions d’accès pour les identités externes

Les utilisateurs qui n’ont plus accès à aucune ressource de votre locataire peuvent être supprimés s’ils ne travaillent plus avec votre organisation. Avant de bloquer et de supprimer ces identités externes, vous souhaiterez peut-être contacter ces utilisateurs externes et vérifier que vous n’avez pas négligé un projet ou un accès permanent dont ils ont encore besoin. Lorsque vous créez un groupe qui contient toutes les identités externes, car vous avez constaté que les membres n’ont accès à aucune ressource de votre locataire, vous pouvez utiliser les révisions d’accès afin que tous les externes attestent eux-mêmes s’ils ont ou non encore besoin de l’accès. Dans le cadre du processus, le créateur de la vérification dans les révisions d’accès peut utiliser la fonction **Exiger la raison lors de l’approbation** pour obliger les utilisateurs externes à fournir une justification de l’accès continu, grâce à laquelle vous pouvez savoir où et comment ils ont encore besoin d’accéder à votre locataire. Vous pouvez également activer la fonctionnalité **Contenu supplémentaire pour l’e-mail du réviseur** , afin d’informer les utilisateurs qu’ils perdront leur accès s’ils ne répondent pas et que, s’ils ont toujours besoin d’un accès, une justification est nécessaire. Si vous souhaitez continuer et laisser les révisions d’accès **désactiver et supprimer** des identités externes dans le cas où les utilisateurs ne répondent pas ou ne fournissent pas de raison valable pour conserver l’accès, vous pouvez utiliser l’option Désactiver et supprimer, comme décrit dans la section suivante.

![limitation de l’étendue de la révision aux utilisateurs invités uniquement](media/access-reviews-external-users/guest-users-only.png)

Une fois la révision terminée, la page **Résultats** affiche une vue d’ensemble de la réponse fournie par chaque identité externe. Vous pouvez choisir d’appliquer les résultats automatiquement et de laisser les révisions d’accès les désactiver et les supprimer. Vous pouvez également examiner les réponses fournies et décider si vous souhaitez supprimer l’accès d’un utilisateur ou le recontacter et obtenir des informations supplémentaires avant de prendre une décision. Si certains utilisateurs ont toujours accès aux ressources que vous n’avez pas encore examinées, vous pouvez utiliser la révision dans le cadre de la détection et enrichir votre prochain cycle de révision et d’attestation.

## <a name="disable-and-delete-external-identities-with-azure-ad-access-reviews-preview"></a>Désactiver et supprimer des identités externes à l’aide des révisions d’accès Azure AD (préversion)

Outre la possibilité de supprimer les identités externes indésirables de ressources telles que les groupes ou les applications, les révisions d’accès Azure AD peuvent empêcher des identités externes de se connecter à votre locataire et supprimer les identités externes de votre locataire après 30 jours. Une fois que vous avez sélectionné **Empêcher l’utilisateur de se connecter pendant 30 jours, puis supprimer l’utilisateur du locataire** , la révision reste dans l’état « application » pendant 30 jours. Durant cette période, les paramètres, les résultats, les réviseurs ou les journaux d’audit dans le cadre de la révision actuelle ne seront pas visibles ou configurables. 

![paramètres une fois l’opération terminée](media/access-reviews-external-users/upon-completion-settings.png)

Lors de la création d’une révision d’accès, dans la section « Paramètres Une fois l’opération terminée », pour **Action à appliquer aux utilisateurs refusés** , vous pouvez définir **Empêcher les utilisateurs de se connecter pendant 30 jours, puis supprimer l’utilisateur du locataire**.
Ce paramètre, actuellement disponible en préversion, vous permet d’identifier, de bloquer et de supprimer des identités externes de votre locataire Azure AD. Les identités externes qui sont révisées et dont l’accès continu est refusé par le réviseur seront bloquées et supprimées, quel que soit l’accès aux ressources ou l’appartenance au groupe dont elles disposent. Il est préférable d’utiliser ce paramètre en dernière étape, après avoir vérifié que les utilisateurs externes en cours de révision n’ont plus accès aux ressources et peuvent être supprimés de votre locataire en toute sécurité ou si vous souhaitez vous assurer qu’ils sont supprimés, quel que soit leur accès permanent. La fonctionnalité « Désactiver et supprimer » bloque d’abord l’utilisateur externe, ce qui l’empêche de se connecter à votre locataire et d’accéder aux ressources. L’accès aux ressources n’est pas révoqué à ce stade et, au cas où vous souhaiteriez réintégrer l’utilisateur externe, sa capacité à se connecter peut être reconfigurée. Si aucune autre action n’est effectuée, une identité externe bloquée sera supprimée du répertoire au bout de 30 jours, ce qui supprimera le compte ainsi que son accès.

## <a name="next-steps"></a>Étapes suivantes

- [Révisions d’accès - API Graph](/graph/api/resources/accessreviews-root?view=graph-rest-beta)
- [Gestion des droits d’utilisation - API Graph](/graph/api/resources/entitlementmanagement-root?view=graph-rest-beta)