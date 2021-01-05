---
title: Autres scénarios de licence basée sur le groupe – Azure AD | Microsoft Docs
description: Plus de scénarios pour la licence basée sur le groupe Azure Active Directory
services: active-directory
keywords: Gestion des licences Azure AD
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.subservice: enterprise-users
ms.topic: how-to
ms.workload: identity
ms.date: 12/02/2020
ms.author: curtand
ms.reviewer: sumitp
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 72a28aebe71809f3e47f9cc6936366cd027a32bb
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96546673"
---
# <a name="scenarios-limitations-and-known-issues-using-groups-to-manage-licensing-in-azure-active-directory"></a>Scénarios, limitations et problèmes connus liés à l’utilisation de groupes dans le cadre de la gestion des licences dans Azure Active Directory

Utilisez les informations et exemples suivants pour approfondir vos connaissances sur la gestion des licences par groupe Azure Active Directory (Azure AD).

## <a name="usage-location"></a>Emplacement d’utilisation

Certains services Microsoft ne sont pas disponibles dans tous les emplacements. Avant de pouvoir attribuer une licence à un utilisateur, l’administrateur doit spécifier la propriété **Emplacement d’utilisation** sur l’utilisateur. Dans le [portail Azure](https://portal.azure.com), vous pouvez spécifier le lieu d’utilisation dans **Utilisateur** &gt; **Profil** &gt; **Paramètres**.

Pour l’affectation d’une licence à un groupe, tous les utilisateurs sans emplacement d’utilisation spécifié héritent de l’emplacement du répertoire. Si vos utilisateurs se trouvent à plusieurs emplacements, assurez-vous de bien prendre en compte cette situation dans vos ressources utilisateur avant d’ajouter des utilisateurs à des groupes avec licences.

> [!NOTE]
> L’affectation d’une licence de groupe n’a jamais pour effet de modifier une valeur d’emplacement d’utilisation existante sur un utilisateur. Nous vous recommandons de toujours définir l’emplacement d’utilisation dans le cadre de votre flux de création d’utilisateur dans Azure AD (par exemple, via une Configuration AAD Connect). Cela garantit que le résultat de l’affectation de licence est toujours correct et que les utilisateurs ne reçoivent pas de services dans des emplacements non autorisés.

## <a name="use-group-based-licensing-with-dynamic-groups"></a>Utilisation de la gestion des licences par groupe avec des groupes dynamiques

Vous pouvez utiliser la gestion des licences par groupe avec n’importe quel groupe de sécurité, ce qui signifie que vous pouvez la combiner avec des groupes dynamiques Azure AD. Les groupes dynamiques exécutent des règles sur les attributs de ressource utilisateur pour ajouter et supprimer des utilisateurs de groupes automatiquement.

Par exemple, vous pouvez créer un groupe dynamique pour un jeu de produits que vous souhaitez attribuer à des utilisateurs. Chaque groupe est rempli par une règle qui ajoute des utilisateurs par leurs attributs, et chaque groupe reçoit les licences que vous décidez de lui attribuer. Vous pouvez affecter l’attribut en local puis le synchroniser avec Azure AD, ou bien vous pouvez gérer l’attribut directement dans le cloud.

Les licences sont attribuées aux utilisateurs peu de temps après leur ajout au groupe. En cas de modification de son attribut, l’utilisateur quitte les groupes et les licences sont supprimées.

### <a name="example"></a> Exemple

Considérez l’exemple d’une solution locale de gestion des identités, qui détermine quels utilisateurs doivent pouvoir accéder aux services web Microsoft. Elle utilise **extensionAttribute1** pour stocker une valeur de chaîne représentant les licences que l’utilisateur doit avoir. Azure AD Connect la synchronise avec Azure AD.

Des utilisateurs peuvent avoir besoin d’une licence mais pas d’une autre, ou avoir besoin des deux. Voici un exemple dans lequel vous distribuez des licences Office 365 Entreprise E5 et Entreprise Mobility + Security à des utilisateurs dans des groupes :

#### <a name="office-365-enterprise-e5-base-services"></a>Office 365 Enterprise E5 : services de base

![Capture d’écran des services de base Office 365 Enterprise E5](./media/licensing-group-advanced/o365-e5-base-services.png)

#### <a name="enterprise-mobility--security-licensed-users"></a>Enterprise Mobility + Security : utilisateurs sous licence

![Capture d’écran des utilisateurs sous licence Enterprise Mobility + Security](./media/licensing-group-advanced/o365-e5-licensed-users.png)

Pour cet exemple, modifiez un utilisateur et définissez son attribut extensionAttribute1 sur la valeur `EMS;E5_baseservices;` si vous souhaitez que l’utilisateur dispose des deux licences. Vous pouvez effectuer cette modification localement. Une fois la modification synchronisée avec le cloud, l’utilisateur est ajouté automatiquement aux deux groupes et les licences sont affectées.

![Capture d’écran montrant comment définir extensionAttribute1 de l’utilisateur](./media/licensing-group-advanced/user-set-extensionAttribute1.png)

> [!WARNING]
> Faites preuve de vigilance lorsque vous modifiez la règle d’appartenance au groupe existante. Lors de la modification d’une règle, l’appartenance du groupe est réévaluée et les utilisateurs qui ne correspondent pas à la nouvelle règle sont supprimés (ceux qui y correspondent ne sont pas affectés par ce processus). Les licences de ces utilisateurs sont supprimées au cours du processus, ce qui peut occasionner une perte de service ou, dans certains cas, une perte de données.
> 
> Si vous avez un grand groupe dynamique dont vous dépendez pour l’attribution des licences, songez à valider des changements majeurs sur un groupe de test plus petit avant de les appliquer au groupe principal.

## <a name="multiple-groups-and-multiple-licenses"></a>Plusieurs groupes et plusieurs licences

Un utilisateur peut être membre de plusieurs groupes avec des licences. Voici quelques possibilités d’opérations à prendre en considération :

- Plusieurs licences pour le même produit peuvent se chevaucher, avec pour effet que tous les services activés sont appliqués à l’utilisateur. L’exemple suivant montre deux groupes de licences : *E3 - Services de base* contient les services de base à déployer en premier, pour tous les utilisateurs. *E3 - Services étendus* contient les services supplémentaires (Sway et Planner) à déployer uniquement pour certains utilisateurs. Dans cet exemple, l’utilisateur a été ajouté aux deux groupes :

  ![Capture d’écran des services activés](./media/licensing-group-advanced/view-enabled-services.png)

  7 des 12 services du produit sont donc activés pour l’utilisateur alors qu’il n’utilise qu’une seule licence pour ce produit.

- La sélection de la licence *E3* montre d’autres détails, notamment des informations sur les services activés pour l’utilisateur par l’affectation des licences de groupe.

## <a name="direct-licenses-coexist-with-group-licenses"></a>Coexistence de licences directes avec des licences de groupe

Quand un utilisateur hérite d’une licence d’un groupe, vous ne pouvez pas supprimer ou modifier directement cette attribution de licence dans les propriétés de l’utilisateur. Les modifications doivent être effectuées dans le groupe et propagées à tous les utilisateurs.

Il est toutefois possible d’affecter la même licence de produit directement à l’utilisateur en plus de la licence héritée. Vous pouvez activer des services supplémentaires à partir du produit pour un utilisateur individuel, sans affecter les autres utilisateurs.

Des licences attribuées directement peuvent être supprimées sans que cela affecte les licences héritées. Prenons l’utilisateur qui hérite d’une licence Office 365 Entreprise E3 d’un groupe.

Initialement, l’utilisateur hérite uniquement de la licence du groupe *E3 - Services de base*, qui active quatre plans de service.

1. Sélectionnez **Affecter** pour affecter directement une licence E3 à l’utilisateur. Dans ce cas, vous désactivez tous les plans de service, à l’exception de Yammer Enterprise.

    Par conséquent, l’utilisateur n’utilise toujours qu’une seule licence du produit E3. L’affectation directe active le service Yammer Enterprise pour cet utilisateur uniquement. Vous pouvez voir les services activés par l’appartenance au groupe et ceux qui le sont par l’affectation directe.

1. Lorsque vous utilisez une affectation directe, les opérations suivantes sont autorisées :

   - Yammer Enterprise peut être désactivé sur la ressource utilisateur directement. Notez que le bouton bascule **Activé/Désactivé** dans l’illustration est activé pour ce service, contrairement aux autres services. Étant donné que le service est activé directement sur l’utilisateur, il est possible de le modifier.
   - Des services supplémentaires peuvent également être activés dans le cadre de la licence affectée directement.
   - Le bouton **Supprimer** peut être utilisé pour supprimer la licence directe de l’utilisateur. Vous pouvez constater que l’utilisateur ne dispose à présent que de la licence de groupe héritée et que seuls les services d’origine restent activés :

## <a name="managing-new-services-added-to-products"></a>Gestion des nouveaux services ajoutés aux produits

Lorsque Microsoft ajoute un nouveau service à un plan de licence de produit, ce service est activé par défaut dans tous les groupes auxquels vous avez attribué la licence du produit. Les utilisateurs de votre organisation qui sont abonnés aux notifications relatives aux modifications apportées au produit reçoivent des e-mails à l’avance, les informant des ajouts de service à venir.

En tant qu’administrateur, vous pouvez passer en revue tous les groupes affectés par la modification et prendre des mesures telles que la désactivation du nouveau service dans tous les groupes. Par exemple, si vous avez créé des groupes ciblant uniquement des services spécifiques pour le déploiement, vous pouvez réexaminer ces groupes pour vous assurer que les nouveaux services ajoutés sont désactivés.

Voici un exemple de ce à quoi ce processus peut ressembler :

1. À l’origine, vous avez affecté le produit *Office 365 Enterprise E5* à plusieurs groupes. L’un de ces groupes, nommé *O365 E5 - Exchange uniquement* a été conçu pour activer uniquement le service *Exchange Online (Plan 2)* pour ses membres.

2. Vous avez reçu de Microsoft une notification signalant que le produit E5 va être étendu avec un nouveau service, *Microsoft Stream*. Lorsque le service devient disponible dans votre organisation, vous pouvez procéder de la façon suivante :

3. Accédez au panneau [**Azure Active Directory > Licences > Tous les produits**](https://portal.azure.com/#blade/Microsoft_AAD_IAM/LicensesMenuBlade/Products), sélectionnez *Office 365 Entreprise E5*, puis **Groupes sous licence** pour afficher la liste de tous les groupes disposant de ce produit.

4. Cliquez sur le groupe que vous souhaitez examiner (dans ce cas, *O365 E5 - Exchange uniquement*). Cela a pour effet d’ouvrir l’onglet **Licences**. Cliquez sur la licence E5 a pour effet d’ouvrir un panneau répertoriant tous les services activés.
   > [!NOTE]
   > Le service *Microsoft Stream* a été automatiquement ajouté et activé dans ce groupe, en plus du service *Exchange Online*:

   ![Capture d’écran du nouveau service ajouté à une licence de groupe](./media/licensing-group-advanced/manage-new-services.png)

5. Si vous souhaitez désactiver le nouveau service dans ce groupe, cliquez sur le bouton bascule **Activé/Désactivé** en regard du service, puis sur le bouton **Enregistrer** pour confirmer la modification. Azure AD va maintenant traiter tous les utilisateurs du groupe pour appliquer la modification. Le service *Microsoft Stream* ne sera pas activé pour les nouveaux utilisateurs ajoutés au groupe.

   > [!NOTE]
   > Il se peut néanmoins que le service soit activé pour ces utilisateurs par le biais d’une autre attribution de licence (un autre groupe dont ils sont membres ou une attribution directe de licence).

6. Si nécessaire, procédez de même pour d’autres groupes auxquels ce produit des attribué.

## <a name="use-powershell-to-see-who-has-inherited-and-direct-licenses"></a>Utiliser PowerShell pour voir qui dispose de licences héritées et directes
Vous pouvez utiliser un script PowerShell pour vérifier si les utilisateurs disposent d’une licence attribuée directement ou héritée d’un groupe.

1. Exécutez l’applet de commande `connect-msolservice` pour vous authentifier et vous connecter à votre organisation.

2. `Get-MsolAccountSku` peut être utilisé pour connaître toutes les licences de produit provisionnées dans l’organisation Azure AD.

   ![Capture d’écran de l’applet de commande Get-Msolaccountsku](./media/licensing-group-advanced/get-msolaccountsku-cmdlet.png)

3. Utilisez avec ce [script PowerShell](licensing-ps-examples.md#check-if-user-license-is-assigned-directly-or-inherited-from-a-group) la valeur *AccountSkuId* correspondant à la licence qui vous intéresse. Cela génère la liste des utilisateurs disposant de cette licence, avec les informations concernant la manière dont la licence a été attribuée.

## <a name="use-audit-logs-to-monitor-group-based-licensing-activity"></a>Utiliser les journaux d’audit pour surveiller l’activité de gestion des licences par groupe

Vous pouvez utiliser les [Journaux d’audit AD Azure](../reports-monitoring/concept-audit-logs.md#audit-logs) pour voir toutes les activités relatives à la gestion des licences par groupe, à savoir :
- qui a modifié les licences attribuées aux groupes ;
- quand le système a démarré le traitement d’une modification de licence de groupe et quand il a terminé ;
- quelles modifications de licence ont été apportées à un utilisateur à la suite d’une attribution de licence de groupe.

>[!NOTE]
> Des journaux d’audit sont disponibles sur la plupart des panneaux dans la section Azure Active Directory du portail. Selon l’emplacement où vous accédez à ceux-ci, des filtres peuvent être préappliqués pour afficher uniquement l’activité pertinente dans le contexte du panneau. Si vous ne voyez pas les résultats attendus, examinez les [options de filtrage](../reports-monitoring/concept-audit-logs.md#filtering-audit-logs) ou accédez aux journaux d’audit non filtrés sous [**Azure Active Directory > Activité > Journaux d’audit**](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Audit).

### <a name="find-out-who-modified-a-group-license"></a>Déterminer qui a modifié une licence de groupe

1. Définissez le filtre **Activité** sur *Définition d’une licence de groupe*, puis cliquez sur **Appliquer**.
2. Les résultats incluent tous les cas de définition ou de modification de licences sur les groupes.
   >[!TIP]
   > Vous pouvez également taper le nom du groupe dans le filtre *Cible* afin de délimiter l’étendue des résultats.

3. Sélectionnez un élément dans la liste pour voir les détails de ce qui a changé. Sous *Propriétés modifiées* sont répertoriées les valeurs tant anciennes que nouvelles d’attribution de licence.

Voici un exemple de modifications récentes de licence groupe, avec des détails :

![Capture d’écran montrant la page « journaux d’audit » avec un élément de liste sélectionné et le volet « journal d’audit des détails d’activité » ouvert.](./media/licensing-group-advanced/audit-group-license-change.png)

### <a name="find-out-when-group-changes-started-and-finished-processing"></a>Déterminer quand le traitement de modifications de groupe a commencé et s’est terminé

Quand une licence change pour un groupe, Azure AD commence à appliquer les modifications à tous les utilisateurs.

1. Pour voir quand le traitement des groupes a commencé, définissez le filtre **Activité** sur *Commencer à appliquer une licence basée sur un groupe à des utilisateurs*. Notez que l’acteur pour l’opération est *Microsoft Azure AD Group-Based Licensing*, un compte système utilisé pour exécuter toutes les modifications de licence de groupe.
   >[!TIP]
   > Cliquez sur un élément dans la liste pour afficher le champ *Propriétés modifiées*. Celui-ci affiche les modifications de licence sélectionnées pour traitement. Cela est utile si vous avez apporté plusieurs modifications à un groupe et n’êtes certain de celles qui ont été traitées.

2. De même, pour voir quand le traitement des groupes s’est terminé, utilisez la valeur de filtre *Terminer l’application d’une licence basée sur le groupe à des utilisateurs*.
   > [!TIP]
   > Dans ce cas, le champ *Propriétés modifiées* contient une synthèse des résultats, qui est utile pour vérifier rapidement si le traitement a généré des erreurs. Exemple de sortie :
   > ```
   > Modified Properties
   > ...
   > Name : Result
   > Old Value : []
   > New Value : [Users successfully assigned licenses: 6, Users for whom license assignment failed: 0.];
   > ```

3. Pour afficher le journal complet indiquant la manière dont un groupe a été traité, y compris toutes les modifications utilisateur, définissez les filtres suivants :
   - **Initié par (acteur)**  : « Microsoft Azure AD Group-Based Licensing ».
   - **Plage de dates** (facultatif) : plage personnalisée indiquant quand, à votre connaissance, le traitement d’un groupe spécifique a commencé et s’est terminé.

Cet exemple de sortie indique le début du traitement, les modifications utilisateur opérées, et la fin du traitement.

![Capture d’écran de modifications de licence de groupe](./media/licensing-group-advanced/audit-group-processing-log.png)

>[!TIP]
> Cliquer sur des éléments liés à une *Modification de la licence d’un utilisateur* a pour effet d’afficher les détails des changements de licence appliqués à chaque utilisateur.

## <a name="deleting-a-group-with-an-assigned-license"></a>Suppression d’un groupe avec une licence attribuée

Il n’est pas possible de supprimer un groupe disposant d’une licence active assignée. Un administrateur peut supprimer un groupe sans réaliser que cela entraînera des retraits de licences à des utilisateurs - Pour cette raison, nous demandons que les licences soient supprimées du groupe avant de pouvoir le supprimer.

Lorsque vous tentez de supprimer un groupe dans le portail Azure, vous pouvez voir une notification d’erreur comme suit : ![Capture d’écran d’un échec de suppression du groupe](./media/licensing-group-advanced/groupdeletionfailed.png)

Accédez à l’onglet **Licences** du groupe et vérifiez si des licences sont attribuées. Si c’est le cas, supprimez ces licences et essayez de nouveau de supprimer le groupe.

Vous pouvez obtenir des erreurs similaires en tentant de supprimer le groupe avec PowerShell ou API Graph. Si vous utilisez un groupe synchronisé localement, Azure AD Connect peut également signaler des erreurs s’il n’arrive pas à supprimer le groupe dans Azure AD. Dans ce cas, veillez à vérifier s’il existe des licences attribuées au groupe et retirez-les d’abord.

## <a name="limitations-and-known-issues"></a>Limitations et problèmes connus

Si vous utilisez la gestion des licences par groupe, il est conseillé de vous familiariser avec la liste suivante des limitations et problèmes connus.

- Actuellement, la gestion des licences par groupe ne prend pas en charge les groupes contenant d’autres groupes. Si vous appliquez une licence à un groupe imbriqué, les licences ne sont appliquées qu’aux membres utilisateurs de premier niveau immédiat du groupe.

- Cette fonctionnalité peut uniquement être utilisée avec les groupes de sécurité et les groupes Microsoft 365 pour lesquels securityEnabled=TRUE.

- Le [centre d’administration Microsoft 365](https://admin.microsoft.com) ne prend actuellement pas en charge la licence basée sur le groupe. Si un utilisateur hérite d’une licence d’un groupe, cette licence apparaît dans le portail d’administration Office comme une licence utilisateur normale. Si vous tentez de modifier ou de supprimer cette licence, le portail renvoie un message d’erreur. Les licences de groupe héritées ne peuvent pas être modifiées directement sur un utilisateur.

- Lorsque des licences sont attribuées ou modifiées pour un groupe de grande taille (par exemple, 100 000 utilisateurs), cela peut affecter les performances. Plus précisément, le volume de modifications générées par Azure AD Automation peut affecter les performances de synchronisation de votre annuaire entre Azure AD et les systèmes locaux.

- Si vous utilisez des groupes dynamiques pour gérer l’appartenance de l’utilisateur, vérifiez que l’utilisateur fait partie du groupe, ce qui est nécessaire pour l’assignation de licences. Si ce n’est pas le cas, [vérifiez l’état de traitement pour la règle d’appartenance](groups-create-rule.md) du groupe dynamique.

- Dans certaines situations à charge élevée, il peut être long de traiter les modifications de licence pour les groupes ou les modifications d’appartenance à des groupes avec des licences existantes. Si vous constatez que vos modifications mettent plus de 24 heures pour traiter un groupe de 60 000 utilisateurs ou moins, [ouvrez un ticket de support](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/supportRequest) pour nous permettre de faire des recherches. 

- L’automatisation de la gestion des licences ne réagit pas automatiquement à tous les types de modifications dans l’environnement. Par exemple, il se peut que vous manquiez de licences et que certains utilisateurs se retrouvent dans un état d’erreur. Pour libérer le nombre de sièges disponibles, vous pouvez supprimer des licences affectées directement d’autres utilisateurs. Le système ne réagit toutefois pas automatiquement à cette modification et ne corrige pas les utilisateurs dans cet état d’erreur.

  Pour résoudre ces types de limitations, vous pouvez accéder au panneau **Groupe** dans Azure AD et cliquer sur **Retraiter**. Cette commande traite tous les utilisateurs de ce groupe et résout les états d’erreur, si cela est possible.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur d’autres scénarios de gestion des licences par le biais des licences basées sur les groupes, consultez :

* [What is group-based licensing in Azure Active Directory? (Présentation des licences basées sur le groupe dans Azure Active Directory)](../fundamentals/active-directory-licensing-whatis-azure-portal.md)
* [Affectation de licences à un groupe dans Azure Active Directory](licensing-groups-assign.md)
* [Identification et résolution des problèmes de licence pour un groupe dans Azure Active Directory](licensing-groups-resolve-problems.md)
* [Migration des utilisateurs individuels sous licence vers les licences basées sur les groupes dans Azure Active Directory](licensing-groups-migrate-users.md)
* [Guide pratique pour migrer des utilisateurs entre des licences de produit à l’aide de la gestion de licences basée sur des groupes dans Azure Active Directory](licensing-groups-change-licenses.md)
