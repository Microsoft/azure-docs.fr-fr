---
title: 'Synchronisation d’Azure AD Connect : Configurer le filtrage | Microsoft Docs'
description: Explique comment configurer le filtrage dans Azure AD Connect Sync.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 880facf6-1192-40e9-8181-544c0759d506
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 03/26/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 595cf2c1dbc105634d33b426c67e5123b9751e6e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95996540"
---
# <a name="azure-ad-connect-sync-configure-filtering"></a>Synchronisation d’Azure AD Connect : Configurer le filtrage
L’utilisation du filtrage vous permet de contrôler les objets de votre annuaire local qui doivent apparaître dans Azure Active Directory (Azure AD). La configuration par défaut concerne l’ensemble des objets présents dans tous les domaines des forêts configurées. En général, il s’agit de la configuration recommandée. Les utilisateurs qui utilisent les charges de travail Microsoft 365, telles qu’Exchange Online et Skype Entreprise, peuvent tirer parti d’une liste d’adresses globale complète pour envoyer des courriers électroniques et appeler tout le monde. La configuration par défaut leur offre la même expérience qu’une implémentation locale d’Exchange ou de Lync.

Toutefois, dans certains cas, vous devez apporter certaines modifications à la configuration par défaut. Voici quelques exemples :

* Vous prévoyez d’utiliser la [topologie multi-annuaire Azure AD](plan-connect-topologies.md#each-object-only-once-in-an-azure-ad-tenant). Vous devez alors appliquer un filtre pour déterminer les objets qui sont synchronisés avec un annuaire Azure AD spécifique.
* Vous exécutez un pilote pour Azure ou Microsoft 365 et souhaitez uniquement disposer d’un sous-ensemble d’utilisateurs dans Azure AD. Dans le petit pilote, il n’est pas impératif de disposer d’une liste d’adresses globale complète pour illustrer la fonctionnalité.
* Vous disposez d’un grand nombre de comptes de service et d’autres comptes non personnels dont vous ne voulez pas dans Azure AD.
* Pour des raisons de conformité, vous ne supprimez aucun compte d’utilisateur local. Vous vous contentez de les désactiver. Toutefois, vous souhaitez qu’Azure AD ne comporte que des comptes actifs.

Cet article explique comment configurer les différents modes de filtrage.

> [!IMPORTANT]
> Microsoft ne prend pas en charge la modification ou l’utilisation de la synchronisation Azure AD Connect en dehors des actions qui sont documentées de façon formelle. Ces actions peuvent entraîner un état de synchronisation Azure AD Connect incohérent ou non pris en charge. Par conséquent, Microsoft ne peut pas fournir de support technique pour ces déploiements.

## <a name="basics-and-important-notes"></a>Principes de base et remarques importantes
Dans Azure AD Connect Sync, vous pouvez activer le filtrage à tout moment. Si vous avez commencé par une configuration de synchronisation d’annuaires par défaut et configuré le filtrage, les objets éliminés par le filtrage ne sont plus synchronisés avec Azure AD. En raison de ce changement, tous les objets présents dans Azure AD précédemment synchronisés puis filtrés sont supprimés d’Azure AD.

Avant d’apporter des modifications au filtrage, veillez à [désactiver la tâche planifiée](#disable-the-scheduled-task) afin de ne pas exporter accidentellement des modifications dont vous n’avez pas encore vérifié l’exactitude.

Étant donné que le filtrage peut supprimer de nombreux objets simultanément, vous devez vous assurer que les nouveaux filtres sont corrects avant de commencer à exporter des modifications vers Azure AD. Une fois que vous avez terminé la procédure de configuration, nous vous recommandons vivement d’exécuter les [étapes de vérification](#apply-and-verify-changes) avant d’exporter et d’effectuer des modifications dans Azure AD.

Pour vous protéger contre la suppression par inadvertance de nombreux objets, la fonctionnalité « [Prévention des suppressions accidentelles](how-to-connect-sync-feature-prevent-accidental-deletes.md) » est activée par défaut. Si vous supprimez de nombreux objets suite à un filtrage (500 par défaut), vous devez exécuter les étapes décrites dans cet article pour que les suppressions soient appliquées à Azure AD.

Si vous utilisez une version antérieure à novembre 2015 ([1.0.9125](reference-connect-version-history.md)), que vous modifiez une configuration de filtre et que vous utilisez une synchronisation de hachage du mot de passe, vous devez déclencher une synchronisation complète de tous les mots de passe une fois la configuration terminée. Pour plus d’informations sur le déclenchement d’une synchronisation complète des mots de passe, consultez la section [Déclencher une synchronisation complète de tous les mots de passe](tshoot-connect-password-hash-synchronization.md#trigger-a-full-sync-of-all-passwords). Si vous utilisez la version 1.0.9125 ou une version ultérieure, l’opération standard de **synchronisation complète** détermine également si les mots de passe doivent être ou non synchronisés et si cette étape supplémentaire n’est plus nécessaire.

Si des objets **utilisateur** ont été accidentellement supprimés d’Azure AD à la suite d’une erreur de filtrage, vous pouvez recréer ces objets utilisateur dans Azure AD en supprimant vos configurations de filtrage. Vous pouvez ensuite resynchroniser vos annuaires. Cette opération restaure les utilisateurs présents dans la corbeille d’Azure AD. Toutefois, vous ne pouvez pas annuler la suppression d’autres types d’objets. Par exemple, si vous supprimez accidentellement un groupe de sécurité et que ce dernier a été utilisé comme liste de contrôle d’accès (ACL, access-control list) d’une ressource, le groupe et ses ACL ne sont pas récupérables.

Azure AD Connect supprime uniquement les objets qu’il a précédemment considérés comme étant compris dans l’étendue. Si certains objets d’Azure AD ont été créés par un autre moteur de synchronisation et ne figurent pas dans l’étendue, l’ajout d’un filtrage ne les supprime pas. Par exemple, si vous démarrez avec un serveur de synchronisation d’annuaires DirSync qui a créé une copie complète de la totalité de votre annuaire dans Azure AD, puis que vous installez un nouveau serveur de synchronisation Azure AD Connect en parallèle avec le filtrage activé dès le départ, Azure AD Connect ne supprime pas les objets supplémentaires créés par DirSync.

Les configurations de filtrage sont conservées lorsque vous effectuez une installation ou une mise à niveau vers une version plus récente d’Azure AD Connect. Avant d’exécuter le premier cycle de synchronisation, il est toujours recommandé de vérifier que la configuration n’a pas été modifiée par inadvertance après une mise à niveau vers une version plus récente.

Si vous disposez de plusieurs forêts, vous devez appliquer les configurations de filtrage décrites dans cet article à chaque forêt (à condition que vous souhaitiez attribuer la même configuration à la totalité des forêts).

### <a name="disable-the-scheduled-task"></a>Désactiver la tâche planifiée
Pour désactiver le planificateur intégré qui déclenche un cycle de synchronisation toutes les 30 minutes, procédez comme suit :

1. Accédez à une invite de commandes PowerShell.
2. Exécutez `Set-ADSyncScheduler -SyncCycleEnabled $False` pour désactiver le planificateur.
3. Apportez les modifications qui sont décrites dans cet article.
4. Exécutez `Set-ADSyncScheduler -SyncCycleEnabled $True` pour réactiver le planificateur.

**Si vous utilisez une version d’Azure AD Connect antérieure à 1.1.105.0**  
Pour désactiver la tâche planifiée qui déclenche un cycle de synchronisation toutes les trois heures, procédez comme suit :

1. Lancez **Planificateur de tâches** à partir du menu **Démarrer**.
2. Directement sous **Bibliothèque du Planificateur de tâches**, trouvez la tâche **Azure AD Sync Scheduler**, cliquez dessus avec le bouton droit, puis sélectionnez **Désactiver**.  
   ![Planificateur de tâches](./media/how-to-connect-sync-configure-filtering/taskscheduler.png)  
3. Vous pouvez désormais apporter des modifications à la configuration et exécuter le moteur de synchronisation manuellement à partir de la console **Synchronization Service Manager**.

Une fois toutes vos modifications de filtrage terminées, n’oubliez pas de revenir et de réactiver la tâche en cliquant sur **Activer**.

## <a name="filtering-options"></a>Options de filtrage
Les types de configurations de filtrage que vous pouvez appliquer à l’outil de synchronisation d’annuaires sont les suivants :

* [**Basé sur un groupe**](#group-based-filtering) : Un filtrage basé sur un seul groupe peut être configuré seulement lors de l’installation initiale à l’aide de l’Assistant Installation.
* [**Basé sur un domaine**](#domain-based-filtering) : Cette option vous permet de sélectionner les domaines à synchroniser avec Azure AD. Vous pouvez également ajouter et supprimer des domaines au niveau de la configuration du moteur de synchronisation lorsque vous apportez des modifications à votre infrastructure locale après avoir installé la synchronisation Azure AD Connect.
* [**Basé sur une unité d’organisation**](#organizational-unitbased-filtering) : Cette option vous permet de sélectionner les unités d’organisation à synchroniser avec Azure AD. Cette option s’applique à tous les types d’objets présents dans les unités d’organisation sélectionnées.
* [**Basé sur un attribut**](#attribute-based-filtering) : Cette option vous permet de filtrer les objets en fonction de valeurs d’attribut sur les objets. Vous pouvez également avoir des filtres différents pour différents types d’objets.

Vous pouvez utiliser plusieurs options de filtrage en même temps. Par exemple, vous pouvez utiliser le filtrage basé sur une unité d’organisation afin de n’inclure des objets que dans une seule unité d’organisation et, dans le même temps, utiliser un filtrage basé sur un attribut pour filtrer davantage les objets. Lorsque vous utilisez plusieurs méthodes de filtrage, les filtres utilisent un « AND » logique entre les filtres.

## <a name="domain-based-filtering"></a>Filtrage basé sur un domaine
Cette section explique comment configurer votre filtre de domaine. Si vous avez ajouté ou supprimé des domaines dans votre forêt après avoir installé Azure AD Connect, vous devez également mettre à jour la configuration du filtrage.

La meilleure façon de modifier le filtrage basé sur un domaine consiste à exécuter l’Assistant Installation et à modifier le [filtrage domaine et unité organisationnelle](how-to-connect-install-custom.md#domain-and-ou-filtering). L’Assistant Installation automatise toutes les tâches qui sont décrites dans cet article.

Vous ne devez suivre ces étapes que si vous ne parvenez pas à exécuter l’Assistant Installation pour une raison quelconque.

Configuration de filtrage basé sur un domaine se compose des étapes suivantes :

1. Sélectionnez les domaines que vous souhaitez inclure dans la synchronisation.
2. Pour chaque domaine ajouté et supprimé, ajustez les profils d'exécution.
3. [Appliquer et vérifier les modifications](#apply-and-verify-changes)

### <a name="select-the-domains-to-be-synchronized"></a>Sélectionner les domaines à synchroniser
Il existe 2 méthodes pour sélectionner les domaines à synchroniser :
    - Utiliser le service de synchronisation
    - Utiliser l’Assistant Azure AD Connect


#### <a name="select-the-domains-to-be-synchronized-using-the-synchronization-service"></a>Sélectionner les domaines à synchroniser à l’aide du service de synchronisation
Pour définir le filtre de domaine, procédez comme suit :

1. Connectez-vous au serveur qui exécute Azure AD Connect Sync en utilisant un compte membre du groupe de sécurité **ADSyncAdmins** .
2. Lancez **Service de synchronisation** à partir du menu **Démarrer**.
3. Sélectionnez **Connecteurs**, puis, dans la liste **Connecteurs**, sélectionnez le connecteur présentant le type **Services de domaine Active Directory**. Dans **Actions**, sélectionnez **Propriétés**.  
   ![Propriétés du connecteur](./media/how-to-connect-sync-configure-filtering/connectorproperties.png)  
4. Cliquez sur **Configurer des partitions d’annuaire**.
5. Dans la liste **Sélectionner des partitions d’annuaire** , sélectionnez et désélectionnez des domaines en fonction de vos besoins. Vérifiez que seules les partitions que vous voulez synchroniser sont sélectionnées.  
   ![Capture d’écran montrant les partitions d’annuaire dans la fenêtre « Propriétés ».](./media/how-to-connect-sync-configure-filtering/connectorpartitions.png)  
   Si vous avez modifié votre infrastructure Active Directory locale et ajouté ou supprimé des domaines dans la forêt, cliquez sur le bouton **Actualiser** pour obtenir une liste mise à jour. Lorsque vous actualisez les données, des informations d’identification vous sont demandées. Fournissez des informations d’identification disposant d’un accès en lecture à Windows Server Active Directory. Il ne s’agit pas nécessairement de l’utilisateur déjà renseigné dans la boîte de dialogue.  
   ![Actualisation requise](./media/how-to-connect-sync-configure-filtering/refreshneeded.png)  
6. Quand vous avez terminé, fermez la boîte de dialogue **Propriétés** en cliquant sur **OK**. Si vous avez supprimé des domaines de la forêt, un message contextuel vous indique qu’un domaine a été supprimé et que la configuration va être nettoyée.
7. Continuez à ajuster les profils d'exécution.

#### <a name="select-the-domains-to-be-synchronized-using-the-azure-ad-connect-wizard"></a>Sélectionner les domaines à synchroniser à l’aide de l’Assistant Azure AD Connect
Pour définir le filtre de domaine, procédez comme suit :

1.  Lancez l’Assistant Azure AD Connect.
2.  Cliquez sur **Configurer**.
3.  Sélectionnez **Personnaliser les options de synchronisation**, puis cliquez sur **Suivant**.
4.  Entrez vos informations d’identification Azure AD.
5.  Dans l’écran **Répertoires connectés**, cliquez sur **Suivant**.
6.  Dans la page **Filtrage par domaine ou unité d’organisation**, cliquez sur **Actualiser**.  Les nouveaux domaines vont apparaître, tandis que les domaines supprimés vont disparaître.
   ![Partitions](./media/how-to-connect-sync-configure-filtering/update2.png)  

### <a name="update-the-run-profiles"></a>Mettre à jour les profils d’exécution
Si vous avez mis à jour votre filtre de domaine, vous devez également procéder à la mise à jour des profils d’exécution.

1. Dans la liste **Connecteurs** , assurez-vous que le connecteur que vous avez modifié à l’étape précédente est sélectionné. Dans **Actions**, sélectionnez **Configurer des profils d’exécution**.  
   ![Profils d’exécution du connecteur 1](./media/how-to-connect-sync-configure-filtering/connectorrunprofiles1.png)  
2. Recherchez et identifiez les profils suivants :
    * Importation complète
    * Synchronisation complète
    * Importation d’écart
    * Synchronisation d’écart
    * Exporter
3. Pour chaque profil, ajustez les domaines **ajoutés** et **supprimés**.
    1. Pour chacun des cinq profils, procédez comme suit pour chaque domaine **ajouté** :
        1. Sélectionnez le profil d’exécution, puis cliquez sur **Nouvelle étape**.
        2. Sur la page **Configurer une étape**, dans le menu déroulant **Type**, sélectionnez le type d’étape portant le même nom que le profil que vous configurez. Cliquez ensuite sur **Suivant**.  
        ![Profils d’exécution du connecteur 2](./media/how-to-connect-sync-configure-filtering/runprofilesnewstep1.png)  
        3. Sur la page **Configuration du connecteur**, dans le menu déroulant **Partition**, sélectionnez le nom du domaine que vous avez ajouté à votre filtre de domaine.  
        ![Profils d’exécution du connecteur 3](./media/how-to-connect-sync-configure-filtering/runprofilesnewstep2.png)  
        4. Pour fermer la boîte de dialogue **Configurer un profil d’exécution**, cliquez sur **Terminer**.
    2. Pour chacun des cinq profils, procédez comme suit pour chaque domaine **supprimé** :
        1. Sélectionnez le profil d’exécution.
        2. Si la **valeur** de l’attribut **Partition** est un identificateur global unique (GUID), sélectionnez l’étape d’exécution, puis cliquez sur **Supprimer l’étape**.  
        ![Profils d’exécution du connecteur 4](./media/how-to-connect-sync-configure-filtering/runprofilesdeletestep.png)  
    3. Vérifiez votre modification. Chaque domaine que vous souhaitez synchroniser doit normalement être répertorié en tant qu’étape dans chacun des profils d’exécution.
4. Pour fermer la boîte de dialogue **Configurer des profils d’exécution**, cliquez sur **OK**.
5.  Pour terminer la configuration, vous devez exécuter une **importation intégrale** et une **synchronisation delta**. Poursuivez votre lecture de la section [Appliquer et vérifier les modifications](#apply-and-verify-changes).

## <a name="organizational-unitbased-filtering"></a>Filtrage basé sur une unité d’organisation
La meilleure façon de modifier le filtrage basé sur une unité d’organisation consiste à exécuter l’Assistant Installation et à modifier le [filtrage domaine et unité organisationnelle](how-to-connect-install-custom.md#domain-and-ou-filtering). L’Assistant Installation automatise toutes les tâches qui sont décrites dans cet article.

Vous ne devez suivre ces étapes que si vous ne parvenez pas à exécuter l’Assistant Installation pour une raison quelconque.

Pour configurer le filtrage basé sur l’unité d’organisation, procédez comme suit :

1. Connectez-vous au serveur qui exécute Azure AD Connect Sync en utilisant un compte membre du groupe de sécurité **ADSyncAdmins** .
2. Lancez **Service de synchronisation** à partir du menu **Démarrer**.
3. Sélectionnez **Connecteurs**, puis, dans la liste **Connecteurs**, sélectionnez le connecteur présentant le type **Services de domaine Active Directory**. Dans **Actions**, sélectionnez **Propriétés**.  
   ![Propriétés du connecteur](./media/how-to-connect-sync-configure-filtering/connectorproperties.png)  
4. Cliquez sur **Configurer des partitions d’annuaire**, sélectionnez le domaine que vous voulez configurer, puis cliquez sur **Conteneurs**.
5. Lorsque vous y êtes invité, fournissez des informations d’identification disposant d’un accès en lecture à votre service d’annuaire Active Directory local. Il ne s’agit pas nécessairement de l’utilisateur déjà renseigné dans la boîte de dialogue.
6. Dans la boîte de dialogue **Sélectionner des conteneurs**, désactivez les unités d’organisation que vous ne voulez pas synchroniser avec l’annuaire cloud, puis cliquez sur **OK**.  
   ![Unités d’organisation dans la boîte de dialogue Sélectionner des conteneurs](./media/how-to-connect-sync-configure-filtering/ou.png)  
   * Le conteneur **Ordinateurs** doit être sélectionné pour vos ordinateurs Windows 10 afin de réussir la synchronisation avec Azure AD. Si les ordinateurs joints à un domaine sont situés dans d’autres unités d’organisation, assurez-vous que ces dernières sont sélectionnées.
   * Le conteneur **ForeignSecurityPrincipals** doit être sélectionné si vous disposez de plusieurs forêts avec approbations. Ce conteneur permet de résoudre l’appartenance au groupe de sécurité inter-forêts.
   * Si vous avez activé la fonctionnalité d’écriture différée des appareils, l’unité d’organisation **RegisteredDevices** doit être sélectionnée. Si vous utilisez une autre fonction d’écriture différée telle que l’écriture différée de groupe, assurez-vous que ces emplacements sont sélectionnés.
   * Sélectionnez une autre unité d’organisation dans laquelle les utilisateurs, iNetOrgPersons, groupes, contacts et ordinateurs sont localisés. Dans l’illustration, toutes ces unités d’organisation se trouvent dans l’unité d’organisation ManagedObjects.
   * Si vous utilisez le filtrage basé sur le groupe, l’unité d’organisation où se trouve le groupe doit être incluse.
   * Notez que vous pouvez spécifier si les nouvelles unités d’organisation ajoutées après la configuration du filtrage sont ou non synchronisées. Pour plus d'informations, consultez la section suivante.
7. Quand vous avez terminé, fermez la boîte de dialogue **Propriétés** en cliquant sur **OK**.
8. Pour terminer la configuration, vous devez exécuter une **importation intégrale** et une **synchronisation delta**. Poursuivez votre lecture de la section [Appliquer et vérifier les modifications](#apply-and-verify-changes).

### <a name="synchronize-new-ous"></a>Synchroniser les nouvelles unités d’organisation
Les nouvelles unités d’organisation qui sont créées après la configuration du filtrage sont synchronisées par défaut. Cet état est indiqué par une case cochée. Vous pouvez également désélectionner certaines unités d’organisation secondaires. Pour obtenir ce comportement, cliquez sur la case jusqu’à ce qu’elle devienne blanche avec une coche bleue (état par défaut). Puis désélectionnez les unités d’organisation secondaires que vous ne souhaitez pas synchroniser.

Si toutes les unités d’organisation secondaires sont synchronisées, la case est blanche avec une coche bleue.  
![Unité d’organisation avec toutes les cases activées](./media/how-to-connect-sync-configure-filtering/ousyncnewall.png)

Si certaines unités d’organisation secondaires ont été désélectionnées, la case est grise avec une coche blanche.  
![Unité d’organisation avec des unités d’organisation secondaires désélectionnées](./media/how-to-connect-sync-configure-filtering/ousyncnew.png)

Avec cette configuration, une unité d’organisation qui a été créée sous ManagedObjects est synchronisée.

L’Assistant d’installation d’Azure AD Connect crée toujours cette configuration.

### <a name="dont-synchronize-new-ous"></a>Ne pas synchroniser les nouvelles unités d’organisation
Vous pouvez configurer le moteur de synchronisation de façon qu’il ne synchronise pas les nouvelles unités d’organisation après la configuration du filtrage. Cet état est indiqué dans l’interface utilisateur par une case grise sans coche. Pour obtenir ce comportement, cliquez sur la case jusqu’à ce qu’elle devienne blanche sans coche. Puis sélectionnez les unités d’organisation secondaires que vous souhaitez synchroniser.

![Unité d’organisation avec la racine non sélectionnée](./media/how-to-connect-sync-configure-filtering/oudonotsyncnew.png)

Avec cette configuration, une unité d’organisation qui a été créée sous ManagedObjects n’est pas synchronisée.

## <a name="attribute-based-filtering"></a>Filtrage par attribut
Avant d’exécuter ces étapes, vérifiez que vous utilisez la version de novembre 2015 ([1.0.9125](reference-connect-version-history.md)) ou une version ultérieure.

> [!IMPORTANT]
>Microsoft recommande de ne pas modifier les règles par défaut créés par **Azure AD Connect**. Si vous voulez modifier la règle, clonez-la, puis désactivez la règle d’origine. Apportez les modifications souhaitées à la règle clonée. Notez qu’en procédant ainsi (désactiver la règle d’origine), vous ne bénéficiez pas des résolutions de bogues ou des fonctionnalités activées via cette règle.

Le filtrage basé sur un attribut est le moyen le plus simple de filtrer des objets. Vous pouvez exploiter les possibilités de [l’approvisionnement déclaratif](concept-azure-ad-connect-sync-declarative-provisioning.md) pour contrôler la quasi-totalité des aspects qui déterminent le moment où un objet est synchronisé avec Azure AD.

Vous pouvez appliquer un filtrage [entrant](#inbound-filtering) d’Active Directory vers le métaverse, et un filtrage [sortant](#outbound-filtering) du métaverse vers Azure AD. Nous vous recommandons d’appliquer le filtrage entrant, car il est le plus simple à gérer. N’utilisez le filtrage sortant que s’il est nécessaire de joindre des objets issus de forêts différentes pour que l’évaluation puisse avoir lieu.

### <a name="inbound-filtering"></a>Filtrage entrant
Le filtrage entrant utilise la configuration par défaut dans laquelle l’attribut métaverse cloudFiltered des objets transmis à Azure AD ne doit pas être défini sur une valeur à synchroniser. Si cet attribut présente la valeur **True**, l’objet n’est pas synchronisé. De par sa conception, cet attribut ne doit pas être défini sur la valeur **False**. Pour vous assurer que les autres règles peuvent apporter une valeur, cet attribut est supposé n’accepter que les valeurs **True** ou **NULL** (absent).

Dans le cadre du filtrage entrant, vous utilisez les possibilités de **l’étendue** pour déterminer les objets à synchroniser ou non. C’est à ce stade que vous réalisez des ajustements en fonction des besoins particuliers de votre organisation. Le module d’étendue dispose d’un **groupe** et d’une **clause** pour déterminer les cas dans lesquels une règle de synchronisation figure dans l’étendue. Un groupe contient une ou plusieurs clauses. Il existe un « AND » logique entre plusieurs clauses, et un « OR » logique entre plusieurs groupes.

Intéressons-nous à un exemple :  
![Capture d’écran montrant un exemple d’ajout de filtres d’étendue.](./media/how-to-connect-sync-configure-filtering/scope.png)  
Voici la lecture qu’il faut en faire **(service = Informatique) OU (service = Ventes et c = US)** .

Dans les exemples et étapes ci-après, vous vous servez de l’objet utilisateur en guise d’exemple, mais vous pouvez l’utiliser pour tous les types d’objets.

Dans les exemples suivants, la valeur de précédence commence à 50. Il peut s’agir de n’importe quel nombre non utilisé, mais la valeur doit être inférieure à 100.

#### <a name="negative-filtering-do-not-sync-these"></a>Filtrage négatif : « ne pas synchroniser »
Dans l’exemple qui suit, vous excluez (ne synchronisez pas) tous les utilisateurs dont l’élément **extensionAttribute15** présente la valeur **NoSync**.

1. Connectez-vous au serveur qui exécute Azure AD Connect Sync en utilisant un compte membre du groupe de sécurité **ADSyncAdmins** .
2. Lancez **Éditeur de règles de synchronisation** à partir du menu **Démarrer**.
3. Assurez-vous que l’option **Entrante** est sélectionnée, puis cliquez sur **Ajouter une nouvelle règle**.
4. Attribuez à la règle un nom descriptif, par exemple «*Entrée depuis AD – Utilisateur DoNotSyncFilter*». Sélectionnez la forêt appropriée, puis sélectionnez **Utilisateur** en tant que **Type d’objet système connecté**, et **Personne** en tant que **Type d’objet de métaverse**. Dans **Type de lien**, sélectionnez **Jointure**. Dans la zone **Précédence**, tapez une valeur actuellement non utilisée par une autre règle de synchronisation (par exemple, 50), puis cliquez sur **Suivant**.  
   ![1 description entrante](./media/how-to-connect-sync-configure-filtering/inbound1.png)  
5. Dans la zone **Filtre d’étendue**, cliquez sur **Ajouter un groupe**, puis sur **Ajouter une clause**. Dans la zone **Attribut**, sélectionnez **ExtensionAttribute15**. Vérifiez que la zone **Opérateur** est définie sur **EQUAL**, puis tapez la valeur **NoSync** dans la zone **Valeur**. Cliquez sur **Suivant**.  
   ![2 étendue entrante](./media/how-to-connect-sync-configure-filtering/inbound2.png)  
6. Laissez les règles de **Jointure**, puis cliquez sur **Suivant**.
7. Cliquez sur **Ajouter une transformation**, définissez la zone **Type de flux** sur **Constante**, puis sélectionnez l’attribut **cloudFiltered** dans la zone **Attribut cible**. Dans la zone de texte **Source**, tapez **True**. Cliquez sur **Ajouter** pour enregistrer la règle.  
   ![3 transformation entrante](./media/how-to-connect-sync-configure-filtering/inbound3.png)
8. Pour terminer la configuration, vous devez exécuter une **synchronisation complète**. Poursuivez votre lecture de la section [Appliquer et vérifier les modifications](#apply-and-verify-changes).

#### <a name="positive-filtering-only-sync-these"></a>Filtrage positif : « synchroniser uniquement ces éléments »
L’expression d’un filtrage positif peut se révéler plus complexe, car vous devez également prendre en compte des objets qui ne sont pas évidents à synchroniser, notamment les salles de conférence. Vous allez également remplacer le filtre par défaut dans la règle prête à l’emploi **In from AD - User Join**. Lorsque vous créez votre filtre personnalisé, veillez à ne pas inclure les objets système critiques, les objets de conflit de réplication, les boîtes aux lettres spéciales et les comptes de service pour Azure AD Connect.

L’option de filtrage positif nécessite deux règles de synchronisation : une (ou plusieurs) règles présentant l’étendue correcte des objets à synchroniser, ainsi qu’une deuxième règle de synchronisation fourre-tout excluant tous les objets qui n’ont pas encore été identifiés en tant qu’objets à synchroniser.

Dans l’exemple suivant, vous synchronisez uniquement les objets utilisateur dont l’attribut « service » a pour valeur **Ventes**:

1. Connectez-vous au serveur qui exécute Azure AD Connect Sync en utilisant un compte membre du groupe de sécurité **ADSyncAdmins** .
2. Lancez **Éditeur de règles de synchronisation** à partir du menu **Démarrer**.
3. Assurez-vous que l’option **Entrante** est sélectionnée, puis cliquez sur **Ajouter une nouvelle règle**.
4. Attribuez à la règle un nom descriptif, par exemple «*Entrée depuis AD - Sync Ventes utilisateur*». Sélectionnez la forêt appropriée, puis sélectionnez **Utilisateur** en tant que **Type d’objet système connecté**, et **Personne** en tant que **Type d’objet de métaverse**. Dans **Type de lien**, sélectionnez **Jointure**. Dans la zone **Précédence**, tapez une valeur actuellement non utilisée par une autre règle de synchronisation (par exemple, 51), puis cliquez sur **Suivant**.  
   ![4 description entrante](./media/how-to-connect-sync-configure-filtering/inbound4.png)  
5. Dans la zone **Filtre d’étendue**, cliquez sur **Ajouter un groupe**, puis sur **Ajouter une clause**. Dans la zone **Attribut**, sélectionnez **department**. Vérifiez que l’opérateur est défini sur **EQUAL**, puis tapez la valeur **Sales** dans la zone **Valeur**. Cliquez sur **Suivant**.  
   ![5 étendue entrante](./media/how-to-connect-sync-configure-filtering/inbound5.png)  
6. Laissez les règles de **Jointure**, puis cliquez sur **Suivant**.
7. Cliquez sur **Ajouter une transformation**, sélectionnez la valeur **Constante** dans la zone **Type de flux**, puis sélectionnez l’attribut **cloudFiltered** dans la zone **Attribut cible**. Dans la zone **Source**, tapez **False**. Cliquez sur **Ajouter** pour enregistrer la règle.  
   ![6 transformation entrante](./media/how-to-connect-sync-configure-filtering/inbound6.png)  
   Il s’agit d’un cas particulier dans lequel vous définissez explicitement cloudFiltered sur la valeur **False**.
8. Nous devons maintenant créer la règle de synchronisation fourre-tout. Donnez à la règle un nom descriptif, par exemple «*Entrée depuis Active Directory - Filtre fourre-tout utilisateur*». Sélectionnez la forêt appropriée, puis sélectionnez **Utilisateur** en tant que **Type d’objet système connecté**, et **Personne** en tant que **Type d’objet de métaverse**. Dans **Type de lien**, sélectionnez **Jointure**. Dans la zone **Précédence**, tapez une valeur actuellement non utilisée par une autre règle de synchronisation (par exemple, 99). Vous avez sélectionné une valeur de précédence supérieure (précédence moindre) à la règle de synchronisation précédente. Toutefois, vous avez également laissé de la place de façon à pouvoir ajouter des règles de synchronisation de filtrage par la suite si vous souhaitez procéder à la synchronisation de services supplémentaires. Cliquez sur **Suivant**.  
   ![7 description entrante](./media/how-to-connect-sync-configure-filtering/inbound7.png)  
9. Laissez l’option **Filtre d’étendue** vide, puis cliquez sur **Suivant**. Un filtre vide indique que la règle doit être appliquée à tous les objets.
10. Laissez les règles de **Jointure**, puis cliquez sur **Suivant**.
11. Cliquez sur **Ajouter une transformation**, sélectionnez la valeur **Constante** dans la zone **Type de flux**, puis sélectionnez l’attribut **cloudFiltered** dans la zone **Attribut cible**. Dans la zone **Source**, tapez **True**. Cliquez sur **Ajouter** pour enregistrer la règle.  
    ![3 transformation entrante](./media/how-to-connect-sync-configure-filtering/inbound3.png)  
12. Pour terminer la configuration, vous devez exécuter une **synchronisation complète**. Poursuivez votre lecture de la section [Appliquer et vérifier les modifications](#apply-and-verify-changes).

En cas de besoin, vous pouvez créer d’autres règles du premier type avec lesquelles inclure davantage d’objets dans la synchronisation.

### <a name="outbound-filtering"></a>Filtrage sortant
Dans certains cas, l’exécution du filtrage n’est requise qu’après la jonction des objets dans le métaverse. Par exemple, pour déterminer si un objet doit être synchronisé, il peut être nécessaire d’examiner l’attribut mail à partir de la forêt de ressources et l’attribut userPrincipalName à partir de la forêt de comptes. Le cas échéant, vous créez le filtrage sur la règle de trafic sortant.

Dans cet exemple, vous modifiez le filtrage afin que seuls les utilisateurs dont les attributs mail et userPrincipalName se terminent par @contoso.com soient synchronisés :

1. Connectez-vous au serveur qui exécute Azure AD Connect Sync en utilisant un compte membre du groupe de sécurité **ADSyncAdmins** .
2. Lancez **Éditeur de règles de synchronisation** à partir du menu **Démarrer**.
3. Sous **Type de règles**, cliquez sur **Sortant**.
4. Selon la version de Connect que vous utilisez, trouvez la règle nommée **Out to Azure AD – User Join** ou **Out to Azure AD - User Join SOAInAD**, puis cliquez sur **Modifier**.
5. Dans la fenêtre contextuelle, sélectionnez **Oui** pour créer une copie de la règle.
6. Sur la page **Description**, redéfinissez la zone **Précédence** sur une valeur inutilisée, telle que 50.
7. Dans la barre de navigation gauche, cliquez sur **Filtre d’étendue**, puis cliquez sur **Ajouter une clause**. Dans la zone **Attribut**, sélectionnez **mail**. Dans la zone **Opérateur**, sélectionnez **ENDSWITH**. Dans **valeur**, tapez **\@contoso.com**, puis cliquez sur **Ajouter une clause**. Dans la zone **Attribut**, sélectionnez **userPrincipalName**. Dans la zone **Opérateur**, sélectionnez **ENDSWITH**. Dans **valeur**, tapez **\@contoso.com**.
8. Cliquez sur **Enregistrer**.
9. Pour terminer la configuration, vous devez exécuter une **synchronisation complète**. Poursuivez votre lecture de la section [Appliquer et vérifier les modifications](#apply-and-verify-changes).

## <a name="apply-and-verify-changes"></a>Appliquer et vérifier les modifications
Une fois que vous avez modifié votre configuration, vous devez l’appliquer aux objets déjà présents dans le système. Il est également possible que des objets ne figurant pas actuellement dans le moteur de synchronisation doivent être traités (et que le moteur de synchronisation doive consulter de nouveau le système source pour en vérifier le contenu).

Si vous avez modifié la configuration à l’aide d’un filtrage par **domaine** ou par **unité d’organisation**, vous devez procéder à une **importation intégrale** suivie d’une **synchronisation delta**.

Si vous avez modifié la configuration en utilisant un filtrage par **attribut**, vous devez exécuter une **synchronisation complète**.

Procédez comme suit :

1. Lancez **Service de synchronisation** à partir du menu **Démarrer**.
2. Sélectionnez **Connecteurs**. Dans la liste **Connecteurs**, sélectionnez le connecteur dont vous avez modifié la configuration précédemment. Dans **Actions**, sélectionnez **Exécuter**.  
   ![Exécution de connecteur](./media/how-to-connect-sync-configure-filtering/connectorrun.png)  
3. Dans **Profils d’exécution**, sélectionnez l’opération mentionnée à la section précédente. Si vous devez exécuter deux actions, n’exécutez la seconde qu’une fois la première terminée. (La colonne **État** correspondant au connecteur sélectionné présente la valeur **Inactif**.)

Après la synchronisation, toutes les modifications sont indexées pour l’exportation. Avant d’apporter les modifications dans Azure AD, il est préférable de vérifier qu’elles sont toutes correctes.

1. Démarrez une invite de commandes, puis accédez à `%ProgramFiles%\Microsoft Azure AD Sync\bin`.
2. Exécutez `csexport "Name of Connector" %temp%\export.xml /f:x`.  
   Le nom du connecteur figure dans le service de synchronisation. Le nom est similaire à « contoso.com – Azure AD » pour Azure AD.
3. Exécutez `CSExportAnalyzer %temp%\export.xml > %temp%\export.csv`.
4. Vous disposez maintenant d’un fichier dans %temp% nommé export.csv qui peuvent être examiné dans Microsoft Excel. Ce fichier contient toutes les modifications sur le point d’être exportées.
5. Apportez les modifications nécessaires aux données ou à la configuration, puis réexécutez ces opérations (importer, synchroniser et vérifier) jusqu’à ce que les modifications sur le point d’être exportées soient conformes à vos attentes.

Lorsque vous êtes satisfait, exportez les modifications vers Azure AD.

1. Sélectionnez **Connecteurs**. Dans la liste **Connecteurs**, sélectionnez le connecteur Azure AD. Dans **Actions**, sélectionnez **Exécuter**.
2. Dans **Profils d’exécution**, sélectionnez **Exporter**.
3. Lorsque vos modifications de configuration suppriment de nombreux objets, une erreur d’exportation s’affiche si leur nombre est supérieur au seuil configuré (par défaut, 500). Le cas échéant, vous devez désactiver provisoirement la fonctionnalité « [Prévention des suppressions accidentelles](how-to-connect-sync-feature-prevent-accidental-deletes.md) ».

À présent, il est temps de réactiver le planificateur.

1. Lancez **Planificateur de tâches** à partir du menu **Démarrer**.
2. Directement sous **Bibliothèque du Planificateur de tâches**, trouvez la tâche **Azure AD Sync Scheduler**, cliquez dessus avec le bouton droit et sélectionnez **Activer**.

## <a name="group-based-filtering"></a>Filtrage de groupe
Vous pouvez configurer le filtrage basé sur un groupe la première fois que vous installez Azure AD Connect à l’aide de [l’installation personnalisée](how-to-connect-install-custom.md#sync-filtering-based-on-groups). Ce filtrage est conçu pour un déploiement pilote dans lequel vous ne souhaitez synchroniser qu’un petit ensemble d’objets. Lorsque vous désactivez le filtrage basé sur un groupe, vous ne pouvez plus le réactiver. L’utilisation du filtrage basé sur un groupe dans une configuration personnalisée n’est *pas prise en charge*. La configuration de cette fonctionnalité n’est possible que par le biais de l’Assistant Installation. Lorsque vous avez terminé votre pilote, utilisez l’une des autres options de filtrage indiquées dans cet article. Si vous utilisez le filtrage basé sur l’unité d’organisation avec le filtrage basé sur les groupes, l’unité d’organisation dans laquelle se trouvent les objets de groupe et de membre doit être incluse.

Lors de la synchronisation de plusieurs forêts Active Directory, vous pouvez configurer le filtrage basé sur les groupes en spécifiant un groupe différent pour chaque connecteur Active Directory. Si vous voulez synchroniser un utilisateur dans une forêt Active Directory, alors que cet utilisateur dispose d’un ou de plusieurs objets correspondants dans d’autres forêts Active Directory, vous devez veiller à ce que l’objet utilisateur et tous ses objets correspondants soient compris dans l’étendue du filtrage basé sur les groupes. Exemples :

* Vous avez un utilisateur dans une forêt qui dispose d’un objet FSP (entité de sécurité externe) correspondant dans une autre forêt. Les deux objets doivent se trouver dans l’étendue de filtrage par groupe. Dans le cas contraire, l’utilisateur n’est pas synchronisé sur Azure AD.

* Vous avez un utilisateur dans une forêt qui dispose d’un compte de ressource correspondant (par exemple, une boîte aux lettres liée) dans une autre forêt. De plus, vous avez configuré Azure AD Connect pour lier l’utilisateur au compte de ressource. Les deux objets doivent se trouver dans l’étendue de filtrage par groupe. Dans le cas contraire, l’utilisateur n’est pas synchronisé sur Azure AD.

* Vous avez un utilisateur dans une forêt qui dispose d’un contact par e-mail correspondant dans une autre forêt. De plus, vous avez configuré Azure AD Connect pour lier l’utilisateur au contact par e-mail. Les deux objets doivent se trouver dans l’étendue de filtrage par groupe. Dans le cas contraire, l’utilisateur n’est pas synchronisé sur Azure AD.


## <a name="next-steps"></a>Étapes suivantes
- Apprenez-en davantage sur la configuration de la [synchronisation Azure AD Connect](how-to-connect-sync-whatis.md).
- Apprenez-en davantage sur [l’intégration de vos identités locales à Azure AD](whatis-hybrid-identity.md).
