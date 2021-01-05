---
title: 'Synchronisation d’Azure AD Connect : modifier la configuration par défaut'
description: Cet article vous guide dans les changements de configuration d’Azure AD Connect Sync.
services: active-directory
author: billmath
manager: daveba
ms.assetid: 7b9df836-e8a5-4228-97da-2faec9238b31
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 08/30/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2044653673da10de59d5ff125da44ac1f89e22f9
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861848"
---
# <a name="azure-ad-connect-sync-make-a-change-to-the-default-configuration"></a>Synchronisation d’Azure AD Connect : modifier la configuration par défaut
L’objectif de cet article est d’expliquer comment apporter des modifications à la configuration par défaut dans la synchronisation Azure Active Directory (Azure AD) Connect. Elle explique pas à pas la procédure pour les scénarios courants. À la fin, vous serez capable d’apporter des modifications simples à votre configuration en fonction de vos propres règles d’entreprise.

> [!WARNING]
> Si vous apportez des modifications aux règles de synchronisation out-of-box par défaut, ces modifications seront remplacées lors de la prochaine mise à jour d'Azure AD Connect, ce qui entraînera des résultats de synchronisation inattendus et potentiellement indésirables.
>
> Les règles de synchronisation out-of-box par défaut ont une empreinte numérique. Si vous apportez une modification à ces règles, l’empreinte numérique ne correspondra plus. Vous risquez de rencontrer des problèmes lorsque vous tenterez d’appliquer une nouvelle version d’Azure AD Connect. Procédez à des modifications uniquement de la façon décrite dans cet article.

## <a name="synchronization-rules-editor"></a>Éditeur de règles de synchronisation
L’éditeur de règles de synchronisation sert à afficher et modifier la configuration par défaut. Il se trouve dans le menu **Démarrer**, sous le groupe **Azure AD Connect**.  
![Menu Démarrer avec l’Éditeur de règles de synchronisation](./media/how-to-connect-sync-change-the-configuration/startmenu2.png)

À l’ouverture de l’éditeur, les règles prêtes à l’emploi par défaut apparaissent.

![Éditeur de règles de synchronisation](./media/how-to-connect-sync-change-the-configuration/sre2.png)

### <a name="navigating-in-the-editor"></a>Navigation dans l’Éditeur
Utilisez les listes déroulantes situées en haut de l’éditeur pour trouver rapidement une règle en particulier. Si, par exemple, vous souhaitez afficher les règles comprenant l’attribut proxyAddresses, vous pouvez modifier ainsi les listes déroulantes :  
![Filtrage SRE](./media/how-to-connect-sync-change-the-configuration/filtering.png)  
Pour réinitialiser le filtrage et charger une nouvelle configuration, appuyez sur la touche F5 du clavier.

Dans le coin supérieur droit se trouve le bouton **Ajouter une nouvelle règle**, qui permet de créer des règles personnalisées.

En bas se trouvent les boutons permettant d’appliquer une règle de synchronisation sélectionnée. Les boutons **Modifier** et **Supprimer** permettent de modifier et de supprimer des règles. Le bouton **Exporter** génère un script PowerShell qui recrée la règle de synchronisation. Cette procédure vous permet de déplacer une règle de synchronisation d’un serveur vers un autre.

## <a name="create-your-first-custom-rule"></a>Création de votre première règle personnalisée
Les modifications les plus courantes portent sur les flux de valeurs d’attributs. Les données de votre répertoire source ne sont pas forcément les mêmes que sur Azure AD. Dans l’exemple de cette section, il faut toujours que le nom donné à un utilisateur ait une casse de *nom propre*.

### <a name="disable-the-scheduler"></a>Désactivation du planificateur
Par défaut, le [planificateur](how-to-connect-sync-feature-scheduler.md) s’exécute toutes les 30 minutes. Faites attention à ce qu’il ne démarre pas pendant que vous effectuez les modifications et que vous résolvez les problèmes de vos nouvelles règles. Pour désactiver temporairement le planificateur, démarrez PowerShell et exécutez `Set-ADSyncScheduler -SyncCycleEnabled $false`.

![Désactivation du planificateur](./media/how-to-connect-sync-change-the-configuration/schedulerdisable.png)  

### <a name="create-the-rule"></a>Création de la règle
1. Cliquez sur **Ajouter une nouvelle règle**.
2. Sur la page **Description**, entrez les informations suivantes :  
   ![Filtrage des règles entrantes](./media/how-to-connect-sync-change-the-configuration/description2.png)  
   * **Name** : donnez un nom descriptif à la règle.
   * **Description** : texte descriptif permettant aux autres utilisateurs de comprendre l’objet de la règle.
   * **Système connecté** : système dans lequel se trouve l’objet. Dans ce cas, sélectionnez le **Connecteur Active Directory**.
   * **Type d’objet de système connecté/métaverse** : sélectionnez respectivement **Utilisateur** et **Personne**.
   * **Type de liaison** : remplacez cette valeur par **Jointure**.
   * **Précédence** : indiquez une valeur unique dans le système. Une valeur numérique inférieure indique une priorité plus élevée.
   * **Balise** : laissez ce champ vide. Seules les règles par défaut de Microsoft doivent contenir une valeur dans cette zone.
3. Sur la page **Scoping Filter** (Filtre d’étendue), entrez **givenName ISNOTNULL**.  
   ![Filtre d’étendue des règles entrantes](./media/how-to-connect-sync-change-the-configuration/scopingfilter.png)  
   Cette section permet de préciser à quels objets la règle s’applique. Si vous la laissez vide, la règle s’appliquera à tous les objets utilisateurs, y compris les salles de conférence, les comptes de service et d’autres objets utilisateurs non humains.
4. Sur la page **Règles de jointure**, laissez le champ vide.
5. Sur la page **Transformations**, définissez le **FlowType** sur **Expression**. Pour **Attribut cible**, sélectionnez **givenName**. Et, pour **Source**, entrez **PCase([givenName])** .
   ![Transformations des règles entrantes](./media/how-to-connect-sync-change-the-configuration/transformations.png)  
   Le moteur de synchronisation respecte la casse aussi bien pour le nom de la fonction que pour celui de l’attribut. Si vous faites une erreur de saisie, un message d’avertissement s’affiche lorsque vous ajoutez la règle. Vous pouvez enregistrer et continuer, mais vous devrez ouvrir à nouveau la règle pour la corriger.
6. Cliquez sur **Ajouter** pour enregistrer la règle.

Votre nouvelle règle personnalisée doit être visible pour les autres règles de synchronisation du système.

### <a name="verify-the-change"></a>Vérification des modifications
Après avoir apporté une nouvelle modification, il est préférable de s’assurer que tout fonctionne comme prévu et qu’aucune erreur n’est générée. Selon le nombre d’objets dont vous disposez, il existe deux façons de procéder :

- exécuter une synchronisation complète de tous les objets ;
- exécuter un aperçu et une synchronisation complète sur un seul objet.

Ouvrez le **Service de synchronisation** dans le menu **Démarrer**. Les étapes décrites dans cette section se trouvent toutes dans cet outil.

**Synchronisation complète de tous les objets**  

   1. Sous Actions, sélectionnez **Connecteurs** en haut de la page. Identifiez le connecteur que vous avez modifié dans la section précédente (dans ce cas, Active Directory Domain Services), puis sélectionnez-le. 
   2. Pour **Actions**, sélectionnez **Exécuter**.
   3. Sélectionnez **Synchronisation complète**, puis **OK**.
   ![Synchronisation complète](./media/how-to-connect-sync-change-the-configuration/fullsync.png)  
   Les objets ne sont pas mis à jour dans le métaverse. Vérifiez vos modifications en examinant l’objet dans le métaverse.

**Aperçu et synchronisation complète d’un seul objet**  

   1. Sous Actions, sélectionnez **Connecteurs** en haut de la page. Identifiez le connecteur que vous avez modifié dans la section précédente (dans ce cas, Active Directory Domain Services), puis sélectionnez-le.
   2. Sélectionnez **Search Connector Space**(Rechercher l’espace de connecteur). 
   3. Utilisez **l’Étendue** pour trouver l’objet que vous souhaitez utiliser dans le but de tester la modification. Sélectionnez l’objet et cliquez sur **Aperçu**. 
   4. Sur le nouvel écran, sélectionnez **Aperçu Validation**.  
   ![Commit preview](./media/how-to-connect-sync-change-the-configuration/commitpreview.png)  
   La modification est maintenant validée dans le métaverse.

**Afficher l’objet dans le métaverse**  

1. Sélectionnez quelques exemples d’objets pour vérifier qu’ils ont la valeur attendue et que la règle s’est bien appliquée. 
2. Sélectionnez **Metaverse Search** (Recherche dans le métaverse) en haut de l’écran. Ajoutez les filtres dont vous avez besoin pour trouver les objets concernés. 
3. Dans les résultats de la recherche, ouvrez un objet. Examinez les valeurs d’attributs, et vérifiez dans la colonne **Règles de synchronisation** que la règle a bien été appliquée comme prévu.  
![Metaverse search](./media/how-to-connect-sync-change-the-configuration/mvsearch.png)  

### <a name="enable-the-scheduler"></a>Activation du planificateur
Si tout fonctionne comme prévu, vous pouvez réactiver le planificateur. À partir de PowerShell, exécutez `Set-ADSyncScheduler -SyncCycleEnabled $true`.

## <a name="other-common-attribute-flow-changes"></a>Autres modifications courantes du flux d’attributs
Dans la section précédente, nous avons vu comment apporter des modifications à un flux d’attributs. Dans cette section, vous trouverez d’autres exemples. Les étapes de création de la règle de synchronisation ont été condensées, mais vous trouverez la procédure complète dans la section précédente.

### <a name="use-an-attribute-other-than-the-default"></a>Utiliser un attribut autre que l’attribut par défaut
Dans ce scénario Fabrikam, il existe une forêt où l’alphabet local est utilisé pour le prénom, le nom de famille et le nom complet. La représentation sous forme de caractères latins de ces attributs est stockée dans les attributs d’extension. Pour créer la liste globale des adresses dans Azure AD et Microsoft 365, l’organisation souhaite utiliser ces attributs.

Avec une configuration par défaut, un objet de la forêt locale ressemble à ceci :  
![Flux d’attributs 1](./media/how-to-connect-sync-change-the-configuration/attributeflowjp1.png)

Pour créer une règle avec d’autres flux d’attributs, procédez comme suit :

1. Ouvrez **l’Éditeur de règles de synchronisation** dans le menu **Démarrer**.
2. En maintenant l’option **Entrant** sélectionnée sur la gauche, cliquez sur le bouton **Ajouter une nouvelle règle**.
3. Attribuez à la règle un nom et une description. Sélectionnez l’instance Active Directory locale et les types d’objets souhaités. Dans **Type de lien**, sélectionnez **Jointure**. Pour **Précédence**, choisissez un nombre qui n’est pas utilisé par une autre règle. Les règles par défaut commencent à 100, donc, il est possible d’utiliser la valeur 50 dans cet exemple.
  ![Flux d’attributs 2](./media/how-to-connect-sync-change-the-configuration/attributeflowjp2.png)
4. Laissez le champ **Filtre d’étendue** vide. (Elle doit s’appliquer à tous les objets utilisateurs de la forêt.)
5. Laissez le champ **Règles de jointure** vide. (C’est la règle prête à l’emploi qui gèrera toutes les jointures.)
6. Dans **Transformations**, créez les flux suivants :  
  ![Flux de valeur d’attribut 3](./media/how-to-connect-sync-change-the-configuration/attributeflowjp3.png)
7. Cliquez sur **Ajouter** pour enregistrer la règle.
8. Accédez à **Synchronization Service Manager**. Sous **Connecteurs**, sélectionnez le connecteur auquel vous avez ajouté la règle. Sélectionnez **Exécuter**, puis **Synchronisation complète**. Une synchronisation complète recalcule tous les objets à partir des règles actives.

Il s’agit du résultat obtenu pour le même objet avec cette règle personnalisée :  
![Flux d’attributs 4](./media/how-to-connect-sync-change-the-configuration/attributeflowjp4.png)

### <a name="length-of-attributes"></a>Longueur des attributs
Les attributs de chaîne sont indexables par défaut ; la longueur maximale est de 448 caractères. Si vous utilisez des attributs de chaîne susceptibles d’en comporter davantage, ajoutez cette ligne au flux de valeur d’attribut :  
`attributeName` <- `Left([attributeName],448)`.

### <a name="changing-the-userprincipalsuffix"></a>Modification de userPrincipalSuffix
L’attribut userPrincipalName dans Active Directory n’est pas toujours connu des utilisateurs et peut ne pas convenir comme ID de connexion. L’Assistant Installation de la synchronisation Azure AD Connect permet de choisir un autre attribut, par exemple, *mail*, mais, dans certains cas, l’attribut doit être calculé.

Par exemple, la société Contoso possède deux annuaires AD, un pour la production et un pour les tests. Elle souhaite que les utilisateurs de son client test utilisent un autre suffixe dans l’ID de connexion :  
`userPrincipalName` <- `Word([userPrincipalName],1,"@") & "@contosotest.com"`.

Dans cette expression, prenons tout ce qui figure à gauche du premier signe @-sign (Word) et concaténons avec une chaîne fixe.

### <a name="convert-a-multi-value-attribute-to-single-value"></a>Convertir un attribut à valeurs multiples en attribut à valeur unique
Dans Active Directory, certains attributs ont plusieurs valeurs dans le schéma, même s’ils semblent être à valeur unique dans Utilisateurs et ordinateurs Active Directory. Citons par exemple l’attribut de description :  
`description` <- `IIF(IsNullOrEmpty([description]),NULL,Left(Trim(Item([description],1)),448))`.

Dans cette expression, si l’attribut a une valeur, prendre le premier élément (*Item*) de l’attribut, supprimer les espaces de début et de fin (*Trim*), puis conserver les 448 premiers caractères (*Left*) de la chaîne.

### <a name="do-not-flow-an-attribute"></a>Ne transmettez pas d'attribut
Pour plus d’informations sur le scénario de cette section, consultez la section [Contrôler le processus de flux d’attributs](concept-azure-ad-connect-sync-declarative-provisioning.md#control-the-attribute-flow-process).

Il existe deux manières de ne pas transmettre un attribut. La première consiste à utiliser l'Assistant Installation pour [supprimer les attributs sélectionnés](how-to-connect-install-custom.md#azure-ad-app-and-attribute-filtering). Cette option fonctionne si vous n'avez jamais synchronisé l'attribut auparavant. Toutefois, si vous avez commencé à synchroniser cet attribut et que vous le supprimez ensuite avec cette fonctionnalité, le moteur de synchronisation cesse de le gérer, et les valeurs existantes restent dans Azure AD.

Si vous souhaitez supprimer la valeur d’un attribut et vous assurer qu’il ne sera pas transmis par la suite, il vous faudra créer une règle personnalisée.

Dans ce scénario Fabrikam, nous nous sommes rendu compte que certains des attributs que nous synchronisons vers le cloud ne devraient pas s’y trouver. Nous souhaitons garantir que ces attributs sont supprimés d’Azure AD.  
![Attributs d’extension erronés](./media/how-to-connect-sync-change-the-configuration/badextensionattribute.png)

1. Créez une règle de synchronisation entrante et remplissez la description.
  ![Descriptions](./media/how-to-connect-sync-change-the-configuration/syncruledescription.png)
2. Créez des flux de valeurs d’attributs avec **Expression** comme **FlowType** et **AuthoritativeNull** comme **Source**. Le littéral **AuthoritativeNull** indique que la valeur devrait être vide dans le métaverse, même si une règle de synchronisation de précédence inférieure essaie de la remplir.
  ![Transformation des attributs d’extension](./media/how-to-connect-sync-change-the-configuration/syncruletransformations.png)
3. Enregistrez la règle de synchronisation. Démarrez le **Service de synchronisation**, recherchez le connecteur et sélectionnez **Exécuter**, puis **Synchronisation complète**. Cette étape permet de recalculer tous les flux d’attributs.
4. Vérifiez que les modifications à exporter sont correctes en parcourant l'espace connecteur.
  ![Suppression progressive](./media/how-to-connect-sync-change-the-configuration/deletetobeexported.png)

## <a name="create-rules-with-powershell"></a>Création de règles avec PowerShell
L’éditeur de règles de synchronisation est adapté lorsque vous avez seulement quelques modifications à apporter. Si vous avez de nombreuses modifications à apporter, PowerShell sera peut-être préférable. Certaines fonctionnalités avancées sont uniquement disponibles avec PowerShell.

### <a name="get-the-powershell-script-for-an-out-of-box-rule"></a>Obtenir le script PowerShell pour une règle out-of-box
Pour voir le script PowerShell qui a créé une règle out-of-box, sélectionnez la règle dans l’éditeur de règles de synchronisation, puis cliquez sur **Exporter**. Cette action vous donne le script PowerShell qui a créé la règle.

### <a name="advanced-precedence"></a>Priorité avancée
Les règles de synchronisation out-of-box commencent avec une valeur de priorité de 100. Si vous avez de nombreuses forêts et que vous devez apporter de nombreuses modifications personnalisées, 99 règles de synchronisation pourraient ne pas suffire.

Vous pouvez indiquer au moteur de synchronisation que vous souhaitez insérer des règles supplémentaires avant les règles prêtes à l’emploi. Pour obtenir ce comportement, procédez comme suit :

1. Marquez la première règle de synchronisation prête à l’emploi (**Entrant à partir d’AD – User Join**) dans l’éditeur de règles de synchronisation, puis sélectionnez **Exporter**. Copiez la valeur d’identificateur SR.  
![PowerShell avant modification](./media/how-to-connect-sync-change-the-configuration/powershell1.png)  
2. Créez la nouvelle règle de synchronisation. Vous pouvez pour cela utiliser l’éditeur de règles de synchronisation. Exportez la règle dans un script PowerShell.
3. Dans la propriété **PrecedenceBefore**, insérez la valeur d’identificateur de la règle prête à l’emploi. Définissez la **priorité** sur **0**. L’attribut d’identificateur doit être unique ; ne réutilisez pas non plus le GUID d’une autre règle. Vérifiez également que la propriété **ImmutableTag** n’est pas définie. Elle ne doit l’être que pour les règles prêtes à l’emploi.
4. Enregistrez le script PowerShell et exécutez-le. Le résultat est que votre règle personnalisée se voit affecter la priorité 100 et que toutes les autres règles out-of-box sont incrémentées.  
![PowerShell après modification](./media/how-to-connect-sync-change-the-configuration/powershell2.png)  

Vous pouvez définir plusieurs règles de synchronisation personnalisées en utilisant la même valeur **PrecedenceBefore** si nécessaire.

## <a name="enable-synchronization-of-usertype"></a>Activer la synchronisation de UserType
Azure AD Connect prend en charge la synchronisation de l’attribut **UserType** pour les objets **Utilisateur** dans la version 1.1.524.0 et les versions ultérieures. Plus précisément, ont été introduites les modifications suivantes :

- Le schéma du type d’objet **Utilisateur** dans le Connecteur Azure AD est étendu à l’attribut UserType, qui est de type chaîne et n’a qu’une seule valeur.
- Le schéma du type d’objet **Personne** dans le métaverse est étendu à l’attribut UserType, qui est de type chaîne et n’a qu’une seule valeur.

Par défaut, l’attribut UserType n’est pas activé pour la synchronisation, car il n’existe aucun attribut UserType correspondant dans l’Active Directory local. Vous devez activer manuellement la synchronisation. Auparavant, prenez connaissance de ce comportement, appliqué par Azure AD :

- Azure AD accepte seulement deux valeurs pour l’attribut UserType : **Membre** et **Invité**.
- Si la synchronisation de l’attribut UserType n’est pas activée dans Azure AD Connect, il est défini sur **Membre** pour les utilisateurs Azure AD créés via la synchronisation d’annuaires.
- Avant la version 1.5.30.0, Azure AD n’autorisait pas la modification par Azure AD Connect de l’attribut UserType sur les utilisateurs Azure AD existants. Dans les versions antérieures, il pouvait uniquement être défini lors de la création des utilisateurs Azure AD, et [changé par le biais de PowerShell](/powershell/module/azuread/set-azureaduser).

Avant d’activer la synchronisation de l’attribut UserType, vous devez déterminer comment il sera dérivé d’Active Directory en local. Voici les approches les plus courantes :

- Vous pouvez désigner un attribut AD local inutilisé (par exemple, extensionAttribute1) à utiliser en tant qu’attribut source. Il doit être de type **chaîne**, avoir une seule valeur et contenir la valeur **Membre** ou **Invité**. 

    Si vous choisissez cette approche, vous devez vous assurer que l’attribut désigné est rempli avec la valeur correcte pour tous les objets utilisateur existants dans Active Directory local qui sont synchronisés avec Azure AD avant d’activer la synchronisation de l’attribut UserType.

- Vous avez également la possibilité de dériver la valeur de l’attribut UserType à partir d’autres propriétés. Par exemple, vous voulez synchroniser tous les utilisateurs en tant **qu’Invités** si leur attribut userPrincipalName AD local se termine par l’élément de domaine <em>@partners.fabrikam123.org</em>. 

    Comme nous l’avons précisé, les versions antérieures d’Azure AD Connect ne peuvent pas modifier l’attribut UserType sur des utilisateurs Azure AD existants. Par conséquent, vous devez vous assurer que la logique que vous avez choisie est cohérente avec la manière dont l’attribut UserType est déjà configuré pour tous les utilisateurs Azure AD existants dans votre client.

Les étapes d’activation de la synchronisation de l’attribut UserType peuvent se résumer comme suit :

1.  Désactiver le planificateur de synchronisation et vérifier qu’aucune synchronisation n’est en cours.
2.  Ajouter l’attribut source au schéma du Connecteur AD local.
3.  Ajouter UserType au schéma du Connecteur Azure AD.
4.  Créer une règle de synchronisation du trafic entrant pour transmettre la valeur de l’attribut à partir d’Active Directory en local.
5.  Créer une règle de synchronisation du trafic sortant pour transmettre la valeur de l’attribut à Azure AD.
6.  Exécuter un cycle de synchronisation complet.
7.  Activer le planificateur de synchronisation.

>[!NOTE]
> Le reste de cette section décrit ces étapes. Ils sont décrits dans le cadre d’un déploiement d’Azure AD avec une topologie de forêt unique et sans règles de synchronisation personnalisées. En présence d’une topologie à forêts multiples, de règles de synchronisation personnalisées ou d’un serveur de gestion intermédiaire, adaptez les étapes en conséquence.

### <a name="step-1-disable-the-sync-scheduler-and-verify-there-is-no-synchronization-in-progress"></a>Étape 1 : Désactiver le planificateur de synchronisation et vérifier qu’aucune synchronisation n’est en cours
Pour éviter d’exporter des modifications indésirables sur Azure AD, veillez à ce qu’aucune synchronisation ne se produise pendant la mise à jour des règles de synchronisation. Pour désactiver le planificateur de synchronisation intégré :

 1. Lancez une session PowerShell sur le serveur Azure AD Connect.
 2. Désactivez la synchronisation planifiée en exécutant la cmdlet `Set-ADSyncScheduler -SyncCycleEnabled $false`.
 3. Ouvrez Synchronization Service Manager en accédant au menu **Démarrer** > **Service de synchronisation**.
 4. Accédez à l’onglet **Opérations** et confirmez qu’aucune opération ne possède l’état *En cours*.

### <a name="step-2-add-the-source-attribute-to-the-on-premises-ad-connector-schema"></a>Étape 2 : Ajouter l’attribut source au schéma du connecteur AD local
Certains attributs Azure AD ne sont pas importés dans l’espace connecteur AD local. Pour ajouter l’attribut source à la liste des attributs importés :

 1. Accédez à l’onglet **Connecteurs** dans Synchronization Service Manager.
 2. Cliquez avec le bouton droit sur le Connecteur AD local, puis sélectionnez **Propriétés**.
 3. Dans la boîte de dialogue contextuelle, accédez à l’onglet **Sélectionner les attributs**.
 4. Assurez-vous que l’attribut source est activé dans la liste d’attributs.
 5. Cliquez sur **OK** pour enregistrer.
![Ajouter l’attribut source au schéma du connecteur AD local](./media/how-to-connect-sync-change-the-configuration/usertype1.png)

### <a name="step-3-add-the-usertype-attribute-to-the-azure-ad-connector-schema"></a>Étape 3 : Ajouter un attribut UserType au schéma du Connecteur Azure AD
Par défaut, l’attribut UserType n’est pas importé dans l’espace Azure AD Connect. Pour ajouter l’attribut UserType à la liste des attributs importés :

 1. Accédez à l’onglet **Connecteurs** dans Synchronization Service Manager.
 2. Cliquez avec le bouton droit sur le **Connecteur Azure AD**, puis sélectionnez **Propriétés**.
 3. Dans la boîte de dialogue contextuelle, accédez à l’onglet **Sélectionner les attributs**.
 4. Vérifiez que l’attribut UserType est activé dans la liste d’attributs.
 5. Cliquez sur **OK** pour enregistrer.

![Ajouter un attribut source au schéma du connecteur Azure AD](./media/how-to-connect-sync-change-the-configuration/usertype2.png)

### <a name="step-4-create-an-inbound-synchronization-rule-to-flow-the-attribute-value-from-on-premises-active-directory"></a>Étape 4 : Créer une règle de synchronisation de trafic entrant pour transmettre la valeur de l’attribut à partir de l’Active Directory local
La règle de synchronisation du trafic entrant permet de transmettre la valeur de l’attribut au métaverse à partir de l’attribut source d’Active Directory en local :

1. Ouvrez Synchronization Service Manager en accédant au menu **Démarrer** > **Éditeur de règles de synchronisation**.
2. Définissez le filtre de recherche **Direction** sur **Entrant**.
3. Cliquez sur **Ajouter une nouvelle règle** pour créer une règle de trafic entrant.
4. Sous l’onglet **Description**, définissez la configuration suivante :

    | Attribut | Valeur | Détails |
    | --- | --- | --- |
    | Nom | *Donnez-lui un nom* | Par exemple, *Entrant à partir d’AD – User UserType*. |
    | Description | *Fournissez une description* |  |
    | Système connecté | *Sélectionnez le connecteur AD local* |  |
    | Type d’objet système connecté | **Utilisateur** |  |
    | Type d’objet métaverse | **Person** |  |
    | Type de lien | **Join** |  |
    | Priorité | *Choisissez une valeur comprise entre 1 et 99* | Les valeurs comprises entre 1 et 99 sont réservées aux règles de synchronisation personnalisées. Ne sélectionnez pas de valeur utilisée par une autre règle de synchronisation. |

5. Accédez à l’onglet **Filtre d’étendue**, puis ajoutez un **groupe de filtres à étendue unique** avec la clause suivante :

    | Attribut | Opérateur | Valeur |
    | --- | --- | --- |
    | adminDescription | NOTSTARTWITH | Utilisateur\_ |

    Le filtre d’étendue spécifie à quels objets AD locaux s’applique cette règle de synchronisation du trafic entrant. Dans cet exemple, nous nous servons du filtre d’étendue utilisé dans la règle de synchronisation prête à l’emploi *Entrant à partir d’AD – User Common*, ce qui permet d’éviter que la règle s’applique à des objets utilisateurs créés avec la fonctionnalité d’écriture différée d’utilisateurs Azure AD. Vous devrez peut-être adapter le filtre d’étendue à votre déploiement Azure AD Connect.

6. Accédez à **l’onglet Transformation**, puis implémentez la règle de transformation souhaitée. Si, par exemple, vous avez désigné un attribut AD local inutilisé (p. ex., extensionAttribute1) en tant qu’attribut UserType source, vous pouvez implémenter un flux de valeur d’attribut direct :

    | Type de flux | Attribut cible | Source | Appliquer une seule fois | Type de fusion |
    | --- | --- | --- | --- | --- |
    | Direct | UserType | extensionAttribute1 | Désactivé | Update |

    Autre exemple : vous souhaitez dériver la valeur de l’attribut UserType à partir d’autres propriétés. Par exemple, vous voulez synchroniser tous les utilisateurs en tant qu’Invités si leur attribut userPrincipalName AD local se termine par l’élément de domaine <em>@partners.fabrikam123.org</em>. Vous pouvez implémenter une expression de ce type :

    | Type de flux | Attribut cible | Source | Appliquer une seule fois | Type de fusion |
    | --- | --- | --- | --- | --- |
    | Expression | UserType | IIF(IsPresent([userPrincipalName]),IIF(CBool(InStr(LCase([userPrincipalName]),"@partners.fabrikam123.org")=0),"Member","Guest"),Error("UserPrincipalName is not present to determine UserType")) | Désactivé | Update |

7. Cliquez sur **Ajouter** pour créer la règle de trafic entrant.

![Créer une règle de synchronisation de trafic entrant](./media/how-to-connect-sync-change-the-configuration/usertype3.png)

### <a name="step-5-create-an-outbound-synchronization-rule-to-flow-the-attribute-value-to-azure-ad"></a>Étape 5 : Créer une règle de synchronisation de trafic sortant pour transmettre la valeur de l’attribut à Azure AD
La règle de synchronisation du trafic sortant permet de transmettre la valeur de l’attribut à l’attribut UserType dans Azure AD à partir du métaverse :

1. Accédez à l’Éditeur de règles de synchronisation.
2. Définissez le filtre de recherche **Direction** sur **Sortante**.
3. Cliquez sur le bouton **Ajouter une nouvelle règle**.
4. Sous l’onglet **Description**, définissez la configuration suivante :

    | Attribut | Valeur | Détails |
    | ----- | ------ | --- |
    | Nom | *Donnez-lui un nom* | Par exemple, *Entrant à partir d’AAD – User UserType*. |
    | Description | *Fournissez une description* ||
    | Système connecté | *Sélectionnez le connecteur AAD* ||
    | Type d’objet système connecté | **Utilisateur** ||
    | Type d’objet métaverse | **Person** ||
    | Type de lien | **Join** ||
    | Priorité | *Choisissez une valeur comprise entre 1 et 99* | Les valeurs comprises entre 1 et 99 sont réservées aux règles de synchronisation personnalisées. Ne sélectionnez pas de valeur utilisée par une autre règle de synchronisation. |

5. Accédez à l’onglet **Filtre d’étendue**, puis ajoutez un **groupe de filtres à étendue unique** avec deux clauses :

    | Attribut | Opérateur | Valeur |
    | --- | --- | --- |
    | sourceObjectType | EQUAL | Utilisateur |
    | cloudMastered | NOTEQUAL | True |

    Le filtre d’étendue spécifie à quels objets Azure AD s’applique cette règle de synchronisation du trafic sortant. Dans cet exemple, nous utilisons le filtre d’étendue de la règle de synchronisation prête à l’emploi *Sortant vers AD – User Identity*. Il empêche que la règle ne s’applique aux objets utilisateurs non synchronisés à partir d’Active Directory en local. Vous devrez peut-être adapter le filtre d’étendue à votre déploiement Azure AD Connect.

6. Accédez à l’onglet **Transformation**, puis implémentez la règle de transformation suivante :

    | Type de flux | Attribut cible | Source | Appliquer une seule fois | Type de fusion |
    | --- | --- | --- | --- | --- |
    | Direct | UserType | UserType | Désactivé | Update |

7. Cliquez sur **Ajouter** pour créer la règle de trafic sortant.

![Créer une règle de synchronisation de trafic sortant](./media/how-to-connect-sync-change-the-configuration/usertype4.png)

### <a name="step-6-run-a-full-synchronization-cycle"></a>Étape 6 : Exécuter un cycle complet de synchronisation
En règle générale, un cycle de synchronisation complet est nécessaire, car nous avons ajouté de nouveaux attributs aux schémas Active Directory et du Connecteur Azure AD, et introduit des règles de synchronisation personnalisées. Il est recommandé de vérifier les modifications avant de les exporter vers Azure AD. 

Vous pouvez procéder comme suit pour vérifier les modifications tandis que vous exécutez manuellement les étapes d’un cycle de synchronisation complète.

1. Exécutez une **Importation intégrale** sur le **Connecteur AD local** :

   1. Accédez à l’onglet **Connecteurs** dans Synchronization Service Manager.
   2. Cliquez avec le bouton droit sur le **Connecteur AD local**, puis sélectionnez **Exécuter**.
   3. Dans la boîte de dialogue contextuelle, sélectionnez **Importation intégrale**, puis cliquez sur **OK**.
   4. Attendez que l'opération se termine.

      > [!NOTE]
      > Vous pouvez ignorer l’importation intégrale sur le Connecteur AD local si l’attribut source figure déjà dans la liste des attributs importés, en d’autres termes, si vous n’avez pas eu à effectuer de modifications au cours de [l’Étape 2 : Ajouter l’attribut source au schéma du Connecteur AD local](#step-2-add-the-source-attribute-to-the-on-premises-ad-connector-schema).

2. Exécutez une **Importation intégrale** sur le **Connecteur Azure AD** :

   1. Cliquez avec le bouton droit sur le **Connecteur Azure AD**, puis sélectionnez **Exécuter**.
   2. Dans la boîte de dialogue contextuelle, sélectionnez **Importation intégrale**, puis cliquez sur **OK**.
   3. Attendez que l'opération se termine.

3. Vérifiez le changement de la règle de synchronisation sur un objet utilisateur existant :

    L’attribut source d’Active Directory en local et l’attribut UserType d’Azure AD ont été importés dans leur espace connecteur respectif. Avant de passer à la synchronisation complète, effectuez un **Aperçu** d’un objet utilisateur existant dans l’espace connecteur AD local. L’attribut source de cet objet doit avoir une valeur définie.
    
    Si **l’Aperçu** réussit et que UserType est rempli dans le métaverse, c’est généralement le signe que les règles de synchronisation sont correctement configurées. Pour plus d’informations sur **l’Aperçu**, consultez la section [Vérifier la modification](#verify-the-change).

4. Exécutez une **Synchronisation complète** sur le **Connecteur AD local** :

   1. Cliquez avec le bouton droit sur le **Connecteur AD local**, puis sélectionnez **Exécuter**.
   2. Dans la boîte de dialogue contextuelle, sélectionnez **Synchronisation complète**, puis cliquez sur **OK**.
   3. Attendez que l'opération se termine.

5. Vérifiez les **Exportations en attente** vers Azure AD :

   1. Cliquez avec le bouton droit sur le connecteur **Azure AD** et sélectionnez **Rechercher dans l’espace connecteur**.
   2. Dans la boîte de dialogue contextuelle **Rechercher dans l’espace connecteur** :

      - Définissez l’**Étendue** sur **En attente d’exportation**.
      - Cochez les trois cases : **Ajouter**, **Modifier** et **Supprimer**.
      - Cliquez sur le bouton **Rechercher** pour obtenir la liste d’objets contenant des modifications à exporter. Pour examiner les modifications apportées à un objet donné, double-cliquez sur celui-ci.
      - Vérifiez que les modifications sont correctes.

6. Exécutez une **Exportation** sur le **Connecteur Azure AD** :

   1. Cliquez avec le bouton droit sur le **Connecteur Azure AD**, puis sélectionnez **Exécuter**.
   2. Dans la boîte de dialogue contextuelle **Exécuter le connecteur**, sélectionnez **Exporter**, puis cliquez sur **OK**.
   3. Attendez que l’exportation vers Azure AD se termine.

> [!NOTE]
> Ces étapes ne comprennent pas la synchronisation complète et l’exportation sur le Connecteur Azure AD, qui ne sont pas obligatoires, car les valeurs d’attributs sont transmises uniquement d’Active Directory en local à Azure AD.

### <a name="step-7-re-enable-the-sync-scheduler"></a>Étape 7 : Réactiver le planificateur de synchronisation
Réactivez le planificateur de synchronisation intégré :

1. Démarrez une session PowerShell.
2. Réactivez la synchronisation planifiée en exécutant la cmdlet `Set-ADSyncScheduler -SyncCycleEnabled $true`.


## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur le modèle de configuration dans [Comprendre l’approvisionnement déclaratif](concept-azure-ad-connect-sync-declarative-provisioning.md).
* En savoir plus sur le langage d’expression dans [Comprendre les expressions d’approvisionnement déclaratif](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md).

**Rubriques de présentation**

* [Synchronisation Azure AD Connect : Comprendre et personnaliser la synchronisation](how-to-connect-sync-whatis.md)
* [Intégration des identités locales dans Azure Active Directory](whatis-hybrid-identity.md)
