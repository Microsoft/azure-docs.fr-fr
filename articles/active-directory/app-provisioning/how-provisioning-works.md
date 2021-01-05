---
title: Comprendre le fonctionnement du provisionnement Azure AD | Microsoft Docs
description: Comprendre le fonctionnement du provisionnement Azure AD
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: conceptual
ms.workload: identity
ms.date: 11/04/2020
ms.author: kenwith
ms.reviewer: arvinh
ms.custom: contperf-fy21q2
ms.openlocfilehash: 07e3ff76886a935bf0b7b5c83052e0e7cd7a0429
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97027422"
---
# <a name="how-provisioning-works"></a>Comment fonctionne le provisionnement

Le provisionnement automatique correspond à la création d’identités utilisateur et de rôles dans les applications cloud auxquelles les utilisateurs ont besoin d’accéder. En plus de créer des identités utilisateur, l’approvisionnement automatique comprend la maintenance et la suppression d’identités utilisateur en cas de modification de l’état ou des rôles. Avant de commencer un déploiement, vous pouvez consulter cet article pour savoir comment fonctionne le provisionnement Azure AD et pour obtenir des recommandations concernant la configuration. 

Le **service de provisionnement Azure AD** attribue des utilisateurs aux applications SaaS et aux autres systèmes en se connectant au point de terminaison d’API de gestion des utilisateurs SCIM 2.0 qui est fourni par chaque fournisseur d’application. Le point de terminaison SCIM permet à Azure AD de créer, de mettre à jour et de supprimer des utilisateurs programmatiquement. Pour les applications sélectionnées, le service d’approvisionnement peut également créer, mettre à jour et supprimer d’autres objets liés à l’identité, tels que les groupes et les rôles. Le canal utilisé pour le provisionnement entre Azure AD et l’application est chiffré à l’aide du chiffrement HTTPS TLS 1.2.


![Service de provisionnement Azure AD](./media/how-provisioning-works/provisioning0.PNG)
*Figure 1 : Le service d’approvisionnement Azure AD*

![Workflow de provisionnement des utilisateurs sortant](./media/how-provisioning-works/provisioning1.PNG)
*Figure 2 : Workflow d’approvisionnement d’utilisateurs « sortant » entre Azure AD et des applications SaaS populaires*

![Workflow de provisionnement des utilisateurs entrant](./media/how-provisioning-works/provisioning2.PNG)
*Figure 3 : Workflow d’approvisionnement d’utilisateurs « entrant » entre des applications de gestion du capital humain populaires, et Azure Active Directory et Windows Server Active Directory*

## <a name="provisioning-using-scim-20"></a>Provisionnement à l’aide de SCIM 2.0

Le service de provisionnement Azure AD utilise le [protocole SCIM 2.0](https://techcommunity.microsoft.com/t5/Identity-Standards-Blog/bg-p/IdentityStandards) pour le provisionnement automatique. Le service se connecte au point de terminaison SCIM de l’application, et il utilise le schéma d’objet utilisateur SCIM et les API REST pour automatiser le provisionnement et le déprovisionnement des utilisateurs et des groupes. Un connecteur de provisionnement SCIM est fourni pour la plupart des applications de la galerie Azure AD. Lorsqu’ils créent des applications pour Azure AD, les développeurs peuvent utiliser l’API de gestion des utilisateurs SCIM 2.0 afin de créer un point de terminaison SCIM qui intègre Azure AD pour le provisionnement. Pour plus d’informations, consultez [Créer un point de terminaison SCIM et configurer l’attribution d’utilisateurs](../app-provisioning/use-scim-to-provision-users-and-groups.md).

Si vous souhaitez demander un connecteur de provisionnement automatique Azure AD pour une application qui n’en est pas pourvue, remplissez une [demande d’application Azure Active Directory](https://aka.ms/aadapprequest).

## <a name="authorization"></a>Autorisation

Des informations d’identification sont nécessaires pour qu’Azure AD puisse se connecter à l’API de gestion des utilisateurs de l’application. Pour configurer l’attribution automatique d’utilisateurs dans une application, vous devez entrer des informations d’identification valides. Pour les applications de la galerie, consultez le tutoriel de l’application afin d’en connaître les différents types d’informations d’identification et les exigences. Pour les applications qui ne proviennent pas de la galerie, vous pouvez consulter la documentation [SCIM](./use-scim-to-provision-users-and-groups.md#authorization-for-provisioning-connectors-in-the-application-gallery) pour comprendre les types d’informations d’identification et les exigences. Dans le portail Azure, vous pouvez tester les informations d’identification en demandant à Azure AD de se connecter à l’API de provisionnement de l’application à l’aide des informations d’identification fournies.

## <a name="mapping-attributes"></a>Mappage d’attributs

Si vous activez l’attribution d’utilisateurs pour une application SaaS tierce, le portail Azure va contrôler ses valeurs d’attributs via des mappages d’attributs. Les mappages déterminent les attributs utilisateur qui circulent entre Azure AD et l’application cible lorsque des comptes d’utilisateur sont provisionnés ou mis à jour.

Il existe un ensemble préconfiguré d’attributs et de mappages d’attributs entre les objets utilisateur Azure AD et les objets utilisateur de chaque application SaaS. Certaines applications gèrent d’autres types d’objets parallèlement aux Utilisateurs, tels que des Groupes.

Lors de la configuration du provisionnement, il est important de vérifier et configurer les mappages d’attributs et les workflows qui définissent les propriétés de l’utilisateur (ou du groupe) passant d’Azure AD à l’application. Vérifiez et configurez la propriété correspondante (**Trouver les objets utilisant cet attribut**) qui est utilisée pour identifier de façon unique et établir une correspondance entre les utilisateurs et groupes des deux systèmes.

Vous pouvez personnaliser les mappages d’attributs par défaut en fonction des besoins de votre organisation. Vous pouvez ainsi modifier ou supprimer des mappages d’attributs existants ou en créer de nouveaux. Pour plus d’informations, consultez [Personnalisation des mappages d’attributs d’attribution d’utilisateurs pour les applications SaaS](./customize-application-attributes.md).

Quand vous configurez l’approvisionnement pour une application SaaS, l’un des types de mappages d’attributs que vous pouvez spécifier est un mappage d’expression. Pour ces mappages, vous devez écrire une expression semblable à un script qui vous permet de transformer les données des utilisateurs dans des formats plus acceptables pour l’application SaaS. Pour plus d’informations, consultez [Écriture d’expressions pour les mappages d’attributs](functions-for-customizing-application-data.md).

## <a name="scoping"></a>Scoping 
### <a name="assignment-based-scoping"></a>Étendue basée sur les attributions

Pour le provisionnement sortant entre Azure AD et une application SaaS, les [attributions d’utilisateurs ou de groupes](../manage-apps/assign-user-or-group-access-portal.md) constituent la méthode la plus courante pour déterminer les utilisateurs qui sont compris dans l’étendue du provisionnement. Étant donné que les attributions d’utilisateurs sont également utilisées pour l’authentification unique, cette même méthode peut être utilisée pour gérer l’accès et le provisionnement. L’étendue basée sur les attributions ne s’applique pas aux scénarios de provisionnement entrant tels que Workday et Successfactors.

* **Groupes.** Avec un plan de licence Azure AD Premium, vous pouvez utiliser des groupes pour autoriser l’accès à une application SaaS intégrée. Ensuite, lorsque l’étendue de provisionnement est définie sur **Synchroniser uniquement les utilisateurs et groupes assignés**, le service de provisionnement Azure AD provisionne ou déprovisionne des utilisateurs selon qu’ils sont membres d’un groupe affecté à l’application. L’objet de groupe n’est pas provisionné, sauf si l’application prend en charge les objets de groupe. Assurez-vous que les groupes attribués à votre application ont la propriété « SecurityEnabled » définie sur « False ».

* **Groupes dynamiques.** Le service d’attribution d’utilisateurs Azure AD peut lire et attribuer des utilisateurs dans des [groupes dynamiques](../enterprise-users/groups-create-rule.md). Gardez à l’esprit les mises en garde et suggestions suivantes :

  * Les groupes dynamiques peuvent impacter les performances du provisionnement de bout en bout entre Azure AD et les applications SaaS.

  * La rapidité à laquelle un utilisateur est provisionné ou déprovisionné dans une application SaaS dépend de la vitesse à laquelle le groupe dynamique auquel il appartient réussit à analyser les changements d’appartenance. Pour plus d’informations sur la vérification de l’état de traitement d’un groupe dynamique, consultez [Vérifier l’état de traitement d’une règle d’appartenance](../enterprise-users/groups-create-rule.md).

  * Lorsqu’un utilisateur perd son appartenance à un groupe dynamique, on parle d’événement de déprovisionnement. Envisagez ce scénario lorsque vous créez des règles pour les groupes dynamiques.

* **Groupes imbriqués.** Le service d’attribution d’utilisateurs Azure AD ne peut pas lire ni attribuer des utilisateurs dans des groupes imbriqués. Le service peut uniquement lire et attribuer les utilisateurs qui se trouvent directement dans un groupe attribué explicitement. Cette limitation concernant les « attributions basées sur les groupes à des applications » affecte également l’authentification unique (voir [Utilisation d’un groupe pour gérer l’accès aux applications SaaS](../enterprise-users/groups-saasapps.md)). Vous devez attribuer explicitement les groupes qui contiennent les utilisateurs à attribuer ou les [inclure dans l’étendue](define-conditional-rules-for-provisioning-user-accounts.md).

### <a name="attribute-based-scoping"></a>Étendue basée sur les attributs 

Vous pouvez utiliser les filtres d’étendue pour définir les règles basées sur des attributs qui précisent quels utilisateurs sont provisionnés dans une application. Cette méthode est couramment utilisée pour le provisionnement entrant, allant des applications HCM à Azure AD et Active Directory. Les filtres d’étendue sont configurés comme parties des mappages d’attributs pour chaque connecteur d’approvisionnement d’utilisateur Azure AD. Pour plus d’informations sur la configuration des filtres d’étendue basés sur les attributs, consultez [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](define-conditional-rules-for-provisioning-user-accounts.md).

### <a name="b2b-guest-users"></a>Utilisateurs B2B invités

Il est possible d’utiliser le service d’attribution d’utilisateurs Azure AD pour attribuer des utilisateurs B2B (ou invités) Azure AD dans des applications SaaS. Toutefois, pour permettre aux utilisateurs B2B de se connecter à l’application SaaS à l’aide d’Azure AD, vous devez configurer la fonctionnalité d’authentification unique basée sur SAML de l’application SaaS d’une manière particulière. Pour plus d’informations sur la configuration des applications SaaS afin d’autoriser les connexions d’utilisateurs B2B, consultez [Configurer des applications SaaS pour B2B Collaboration](../external-identities/configure-saas-apps.md).

Notez que le paramètre userPrincipalName d’un utilisateur invité est souvent stocké en tant que « alias#EXT#@domain.com ». Lorsque userPrincipalName est inclus dans vos mappages d’attributs en tant qu’attribut source, la partie #EXT# est supprimée du userPrincipalName. Si vous avez besoin que la partie #EXT# soit présente, remplacez userPrincipalName par originalUserPrincipalName en tant qu’attribut source. 

## <a name="provisioning-cycles-initial-and-incremental"></a>Cycles de provisionnement : cycle initial et cycle incrémentiel

Lorsqu’Azure AD est le système source, le service d’approvisionnement utilise la fonctionnalité [Utilisation des requêtes différentielles pour suivre les modifications apportées aux données de Microsoft Graph](/graph/delta-query-overview) pour surveiller les utilisateurs et groupes. Le service de provisionnement exécute un cycle initial sur le système source et le système cible, qui est suivi de cycles incrémentiels périodiques.

### <a name="initial-cycle"></a>Cycle initial

Une fois le service de provisionnement démarré, le premier cycle va :

1. Interroger tous les utilisateurs et groupes à partir du système source, afin de récupérer tous les attributs définis dans les [mappages d’attributs](customize-application-attributes.md).

2. Filtrer les utilisateurs et les groupes renvoyés à l’aide [d’affectations](../manage-apps/assign-user-or-group-access-portal.md) ou de [filtres d’étendue basés sur un attribut](define-conditional-rules-for-provisioning-user-accounts.md) configurés.

3. Quand un utilisateur est affecté ou se trouve dans une étendue pour l’approvisionnement, le service interroge le système cible pour rechercher un utilisateur correspondant à l’aide des [attributs de correspondance](customize-application-attributes.md#understanding-attribute-mapping-properties) spécifiés. Exemple : si le nom userPrincipal dans le système source est l’attribut de correspondance et s’il est mappé à userName dans le système cible, le service d’approvisionnement interroge le système cible pour les userNames qui correspondent aux valeurs de nom userPrincipal dans le système source.

4. Si aucun utilisateur correspondant n’est trouvé dans le système cible, il est créé à l’aide des attributs renvoyés depuis le système source. Une fois que le compte d’utilisateur a été créé, le service de provisionnement détecte et met en cache l’ID du nouvel utilisateur dans le système cible. Cet ID sera utilisé pour exécuter toutes les futures opérations concernant cet utilisateur.

5. Si un utilisateur correspondant est trouvé, il est mis à jour à l’aide des attributs fournis par le système source. Une fois que le compte d’utilisateur a été trouvé, le service de provisionnement détecte et met en cache l’ID du nouvel utilisateur dans le système cible. Cet ID sera utilisé pour exécuter toutes les futures opérations concernant cet utilisateur.

6. Si les mappages d’attributs contiennent des attributs de « référence », le service effectue des mises à jour supplémentaires sur le système cible pour créer et lier les objets référencés. Par exemple, un utilisateur peut avoir un attribut « Manager » dans le système cible, qui est lié à un autre utilisateur créé dans le système cible.

7. Conserver un filigrane à la fin du cycle initial, qui fournit le point de départ pour les cycles incrémentiels ultérieurs.

Certaines applications telles que ServiceNow, G Suite et Box prennent non seulement en charge l’approvisionnement des utilisateurs, mais également celui des groupes et de leurs membres. Dans ce type de situation, si l’approvisionnement des groupes est activé dans les [mappages](customize-application-attributes.md), le service d’approvisionnement synchronise les utilisateurs et les groupes, puis les appartenances au groupe.

### <a name="incremental-cycles"></a>Cycles incrémentiels

Après le cycle initial, tous les autres cycles effectuent les opérations suivantes :

1. Interroger le système source pour rechercher les utilisateurs et les groupes qui ont été mis à jour depuis le dernier filigrane enregistré.

2. Filtrer les utilisateurs et les groupes renvoyés à l’aide [d’affectations](../manage-apps/assign-user-or-group-access-portal.md) ou de [filtres d’étendue basés sur un attribut](define-conditional-rules-for-provisioning-user-accounts.md) configurés.

3. Quand un utilisateur est affecté ou se trouve dans une étendue pour l’approvisionnement, le service interroge le système cible pour rechercher un utilisateur correspondant à l’aide des [attributs de correspondance](customize-application-attributes.md#understanding-attribute-mapping-properties) spécifiés.

4. Si aucun utilisateur correspondant n’est trouvé dans le système cible, il est créé à l’aide des attributs renvoyés depuis le système source. Une fois que le compte d’utilisateur a été créé, le service de provisionnement détecte et met en cache l’ID du nouvel utilisateur dans le système cible. Cet ID sera utilisé pour exécuter toutes les futures opérations concernant cet utilisateur.

5. Si un utilisateur correspondant est trouvé, il est mis à jour à l’aide des attributs fournis par le système source. Si le compte d’utilisateur trouvé a été récemment attribué, le service de provisionnement détecte et met en cache l’ID du nouvel utilisateur dans le système cible. Cet ID sera utilisé pour exécuter toutes les futures opérations concernant cet utilisateur.

6. Si les mappages d’attributs contiennent des attributs de « référence », le service effectue des mises à jour supplémentaires sur le système cible pour créer et lier les objets référencés. Par exemple, un utilisateur peut avoir un attribut « Manager » dans le système cible, qui est lié à un autre utilisateur créé dans le système cible.

7. Si un utilisateur qui se trouvait précédemment dans l’étendue de provisionnement est supprimé de l’étendue (ou déprovisionné), le service désactive l’utilisateur dans le système cible via une mise à jour.

8. Si un utilisateur qui se trouvait précédemment dans l’étendue de l’approvisionnement est désactivé ou supprimé de façon réversible dans le système source, le service désactive l’utilisateur dans le système cible via une mise à jour.

9. Si un utilisateur qui se trouvait précédemment dans l’étendue de l’approvisionnement est définitivement supprimé du système source, le service efface l’utilisateur dans le système cible. Dans Azure AD, les utilisateurs sont supprimés définitivement 30 jours après leur suppression de façon réversible.

10. Conserver un nouveau filigrane à la fin du cycle initial, qui fournit le point de départ pour les cycles incrémentiels ultérieurs.

> [!NOTE]
> Vous pouvez éventuellement désactiver les opérations de **création**, **mise à jour** et **suppression** à l’aide des cases à cocher **Actions d’objet cible** dans la section [Mappages](customize-application-attributes.md). La logique pour désactiver un utilisateur pendant une mise à jour est également contrôlée via un mappage d’attributs à partir d’un champ tel que « accountEnabled ».

Le service de provisionnement continue à exécuter indéfiniment des cycles incrémentiels consécutifs, à des intervalles définis dans le [tutoriel propre à chaque application](../saas-apps/tutorial-list.md). Les cycles incrémentiels se poursuivent jusqu’à ce que l’un des événements suivants se produise :

- Le service est arrêté manuellement à l’aide du Portail Azure ou à l’aide de la commande appropriée de l’API Microsoft Graph.
- Un nouveau cycle initial est déclenché au moyen de l’option **Effacer l’état en cours et redémarrer** dans le Portail Azure ou à l’aide de la commande appropriée de l’API Microsoft Graph. Cette action permet d’effacer les filigranes stockés et de rendre tous les objets source disponibles pour une nouvelle évaluation.
- Une nouveau cycle initial est déclenché en raison d’une modification dans les mappages d’attributs ou les filtres d’étendue. Cette action permet également d’effacer les filigranes stockés et de rendre tous les objets source disponibles pour une nouvelle évaluation.
- Le processus d’approvisionnement est mis en quarantaine (voir ci-dessous) en raison d’un taux d’erreur élevé et reste en quarantaine pendant plus de quatre semaines. Dans ce cas, le service sera automatiquement désactivé.

### <a name="errors-and-retries"></a>Erreurs et nouvelles tentatives

Si un utilisateur ne peut pas être ajouté, mis à jour ou supprimé dans le système cible en raison d’une erreur dans le système cible, une nouvelle tentative est effectuée lors du prochain cycle de synchronisation. Si l’utilisateur continue d’échouer, les nouvelles tentatives ne seront possibles qu’à une fréquence réduite, pour finalement n’être autorisées qu’une seule fois par jour. Pour résoudre le problème, les administrateurs doivent vérifier les [journaux de provisionnement](../reports-monitoring/concept-provisioning-logs.md?context=azure/active-directory/manage-apps/context/manage-apps-context) pour déterminer la cause racine et prendre les mesures nécessaires. Les échecs courants peuvent inclure :

- Utilisateurs n’ayant pas d’attribut renseigné dans le système source alors qu’il est requis dans le système cible
- Utilisateurs ayant une valeur d’attribut dans le système source pour laquelle il existe une contrainte unique dans le système cible, et la même valeur se trouve dans un autre enregistrement utilisateur

Ces échecs peuvent être résolus en ajustant les valeurs d’attribut de l’utilisateur concerné dans le système source, ou en ajustant les mappages d’attributs pour ne pas provoquer de conflits.

### <a name="quarantine"></a>Mise en quarantaine

Si la plupart ou la totalité des appels effectués sur le système cible échouent constamment en raison d’une erreur (des informations d’identification non valides, par exemple), le travail de provisionnement est mis en quarantaine. Cet état est indiqué dans le [rapport de synthèse sur l’approvisionnement](./check-status-user-account-provisioning.md) et via les notifications par e-mail si elles ont été configurées dans le portail Azure.

Lors de la mise en quarantaine, la fréquence des cycles incrémentiels est progressivement réduite à une fois par jour.

Le travail de provisionnement sort de quarantaine lorsque toutes les erreurs ont été corrigées et que le prochain cycle de synchronisation démarre. Si le travail d’approvisionnement reste en quarantaine pendant plus de quatre semaines, celui-ci est désactivé. Pour en savoir plus sur l’état de quarantaine, [cliquez ici](./application-provisioning-quarantine-status.md).

### <a name="how-long-provisioning-takes"></a>Durée du provisionnement

Les performances varient selon que votre tâche d’approvisionnement exécute un cycle d’approvisionnement initial ou un cycle incrémentiel. Pour en savoir plus sur la durée de l’approvisionnement et comment surveiller l’était du service d’approvisionnement, consultez [Vérifier l’état de l’approvisionnement d’utilisateurs](application-provisioning-when-will-provisioning-finish-specific-user.md).

### <a name="how-to-tell-if-users-are-being-provisioned-properly"></a>Comment savoir si les utilisateurs sont correctement attribués ?

Toutes les opérations effectuées par le service de provisionnement d’utilisateurs sont enregistrées dans les [Journaux de provisionnement (préversion)](../reports-monitoring/concept-provisioning-logs.md?context=azure/active-directory/manage-apps/context/manage-apps-context) Azure AD. Les journaux comprennent toutes les opérations de lecture et d’écriture effectuées sur les systèmes sources et cibles, ainsi que les données utilisateur qui ont été lues ou écrites lors de chaque opération. Pour plus d’informations sur la lecture des journaux de provisionnement dans le portail Azure, consultez le [guide de création de rapports sur le provisionnement](./check-status-user-account-provisioning.md).

## <a name="de-provisioning"></a>Déprovisionnement
Le service d’approvisionnement Azure AD effectue la synchronisation des systèmes sources et cibles en déprovisionnant les comptes lorsque l’accès utilisateur est supprimé.

Le service d’approvisionnement prend en charge la suppression et la désactivation (parfois appelées « suppressions réversibles ») des utilisateurs. La définition exacte de désactivation et de suppression varie en fonction de l’implémentation de l’application cible, mais en général, une désactivation indique que l’utilisateur ne peut pas se connecter. Une suppression indique que l’utilisateur a été entièrement supprimé de l’application. Pour les applications SCIM, une désactivation correspond une demande de définition de la propriété *active* sur false pour un utilisateur. 

**Configurer votre application pour désactiver un utilisateur**

Vérifiez que vous avez activé la case à cocher des mises à jour.

Vérifiez que le mappage est *actif* pour votre application. Si vous utilisez une application à partir de la galerie d’applications, le mappage peut être légèrement différent. Veillez à utiliser le mappage par défaut/prêt à l’emploi pour les applications de la galerie.

:::image type="content" source="./media/how-provisioning-works/disable-user.png" alt-text="Désactiver un utilisateur" lightbox="./media/how-provisioning-works/disable-user.png":::


**Configurer votre application pour supprimer un utilisateur**

Les scénarios suivants déclenchent une désactivation ou une suppression : 
* Un utilisateur est supprimé de manière réversible dans Azure AD (envoyé à la corbeille/ propriété AccountEnabled définie sur false).
    30 jours après sa suppression d’Azure AD, l’utilisateur est définitivement supprimé du locataire. À ce stade, le service de provisionnement envoie une demande de suppression afin de supprimer définitivement l’utilisateur de l’application. Pendant cette période de 30 jours, vous pouvez à tout moment [supprimer manuellement un utilisateur](../fundamentals/active-directory-users-restore.md), ce qui a pour effet d’envoyer une demande de suppression à l’application.
* Un utilisateur est définitivement supprimé / supprimé de la corbeille dans Azure AD.
* Un utilisateur n’est plus affecté à une application.
* Un utilisateur dans l’étendue d’application sort de cette étendue (et ne transmet plus de filtre d’étendue).

:::image type="content" source="./media/how-provisioning-works/delete-user.png" alt-text="Suppression d’un utilisateur" lightbox="./media/how-provisioning-works/delete-user.png":::

Par défaut, le service de provisionnement Azure AD supprime de manière réversible ou désactive les utilisateurs qui sortent de l’étendue. Si vous souhaitez substituer ce comportement par défaut, vous pouvez définir un indicateur permettant d'[ignorer les suppressions non comprises dans l'étendue.](skip-out-of-scope-deletions.md)

Si l’un des quatre événements ci-dessus se produit et si l’application cible ne prend pas en charge les suppressions réversibles, le service de provisionnement envoie une demande de suppression afin de supprimer définitivement l’utilisateur de l’application.

Si vous voyez un attribut IsSoftDeleted dans vos mappages d’attributs, sachez qu’il sera utilisé pour déterminer l’état de l’utilisateur et s’il faut envoyer une demande de mise à jour avec active = false pour supprimer l’utilisateur de manière réversible.

**Limitations connues**

* Si un utilisateur précédemment managé par le service d’approvisionnement n’est plus affecté à une application ou à un groupe affecté à une application, nous envoyons une requête de désactivation. À ce stade, l’utilisateur n’est pas managé par le service et nous n’envoyons aucune requête de suppression lorsqu’il est supprimé de l’annuaire.
* L’approvisionnement d’un utilisateur qui est désactivé dans Azure AD n’est pas pris en charge. Pour pouvoir être approvisionné, l’utilisateur doit être dans Azure AD préalablement.
* Quand un utilisateur supprimé de façon réversible devient actif, le service d’approvisionnement Azure AD l’active dans l’application cible, mais ne restaure pas automatiquement les appartenances aux groupes. L’application cible doit conserver les appartenances aux groupes de l’utilisateur dans un état inactif. Si l’application cible ne prend pas en charge ce comportement, vous pouvez redémarrer l’approvisionnement pour mettre à jour les appartenances aux groupes. 

**Recommandation**

Lors du développement d’une application, prenez toujours en charge les suppressions réversibles et les suppressions définitives. Cela permet aux clients de récupérer un client qui a été désactivé accidentellement.


## <a name="next-steps"></a>Étapes suivantes

[Planifier un déploiement d’attribution automatique d’utilisateurs](../app-provisioning/plan-auto-user-provisioning.md)

[Configurer le provisionnement pour une application de galerie](./configure-automatic-user-provisioning-portal.md)

[Créer un point de terminaison SCIM et configurer le provisionnement lors de la création de votre propre application](../app-provisioning/use-scim-to-provision-users-and-groups.md)

[Résoudre les problèmes liés à la configuration et à l’attribution des utilisateurs dans une application](./application-provisioning-config-problem.md)