---
title: Comment résoudre les règles par défaut modifiées - Azure AD Connect | Microsoft Docs
description: Découvrez comment résoudre les règles par défaut modifiées qui sont fournies avec Azure AD Connect.
services: active-directory
author: billmath
manager: daveba
editor: curtand
ms.reviewer: darora10
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/21/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: a0fc1bc3158e04c9b1f677af7ef2375ac3ed2ce7
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91320045"
---
# <a name="fix-modified-default-rules-in-azure-ad-connect"></a>Résoudre les règles par défaut modifiées dans Azure AD Connect

Azure Active Directory (Azure AD) Connect utilise des règles par défaut pour la synchronisation.  Malheureusement, ces règles ne s’appliquent pas à toutes les organisations. Selon vos besoins, vous devrez peut-être les modifier. Cet article décrit deux exemples parmi les personnalisations les plus courantes et explique la méthode correcte pour les effectuer.

>[!NOTE] 
> La modification des règles par défaut existantes pour atteindre une personnalisation nécessaire n’est pas prise en charge. Si vous procédez ainsi, ces règles ne peuvent pas être mises à jour vers la dernière version dans les futures versions. Vous n’obtiendrez pas les correctifs de bogues dont vous avez besoin, ni les nouvelles fonctionnalités. Ce document explique comment obtenir le même résultat sans modifier les règles par défaut. 

## <a name="how-to-identify-modified-default-rules"></a>Comment identifier les règles par défaut modifiées
Depuis la version 1.3.7.0 d’Azure AD Connect, il est facile d’identifier les règles par défaut modifiées. Accédez à **Applications sur le bureau**, puis sélectionnez **Éditeur de règles de synchronisation**.

![Azure AD Connect, avec l’éditeur de règles de synchronisation mis en surbrillance](media/how-to-connect-fix-default-rules/default1.png)

Dans l’éditeur, toutes les règles par défaut modifiées sont affichées avec une icône d’avertissement devant leur nom.

![Icône d'avertissement](media/how-to-connect-fix-default-rules/default2.png)

 Une règle désactivée avec le même nom à côté apparaît également (il s’agit de la règle par défaut standard).

![L’éditeur de règles de synchronisation, montrant la règle par défaut standard et la règle par défaut modifiée](media/how-to-connect-fix-default-rules/default2a.png)

## <a name="common-customizations"></a>Personnalisations courantes
Les personnalisations courantes pour les règles par défaut sont les suivantes :

- Modifier un flux d’attribut
- Modifier un filtre d’étendue
- Modifier une condition de jointure

Avant de modifier une règle :

- Désactivez le planificateur de synchronisation. Par défaut, le planificateur s’exécute toutes les 30 minutes. Veillez à ce qu’il ne démarre pas pendant que vous effectuez les modifications et que vous résolvez les problèmes de vos nouvelles règles. Pour désactiver temporairement le planificateur, démarrez PowerShell et exécutez `Set-ADSyncScheduler -SyncCycleEnabled $false`.
 ![Commandes PowerShell pour désactiver le planificateur de synchronisation](media/how-to-connect-fix-default-rules/default3.png)

- La modification du filtre d’étendue peut entraîner la suppression des objets dans le répertoire cible. Soyez prudent avant d’apporter des modifications dans l’étendue des objets. Nous vous recommandons d’apporter des modifications à un serveur intermédiaire avant d’apporter des modifications sur le serveur actif.
- Exécuter un aperçu sur un seul objet, comme indiqué dans la section [Valider une règle de synchronisation](#validate-sync-rule), après avoir ajouté une nouvelle règle.
- Exécuter une synchronisation complète après avoir ajouté une nouvelle règle ou avoir modifié une règle de synchronisation personnalisée. Cette synchronisation applique de nouvelles règles à tous les objets.

## <a name="change-attribute-flow"></a>Modifier un flux d’attribut
Il existe trois scénarios différents pour modifier le flux d’attributs :
- Ajout d’un nouvel attribut.
- Remplacement de la valeur d’un attribut existant.
- Choisir de ne pas synchroniser un attribut existant.

Vous pouvez effectuer ces actions sans modifier les règles par défaut standards.

### <a name="add-a-new-attribute"></a>Ajouter un nouvel attribut
Si vous trouvez qu’un attribut n’est pas transmis à partir de votre répertoire source vers le répertoire cible, utilisez la [synchronisation Azure AD Connect : Extensions d’annuaire](how-to-connect-sync-feature-directory-extensions.md) pour résoudre cela.

Si les extensions ne fonctionnent pas pour vous, essayez d’ajouter deux nouvelles règles de synchronisation, décrites dans les sections suivantes.


#### <a name="add-an-inbound-sync-rule"></a>Ajouter une règle de synchronisation entrante
Une règle de synchronisation entrante signifie que la source de l’attribut est un espace de connecteur et la cible est le métaverse. Par exemple, pour avoir un nouveau flux d’attribut à partir d’Active Directory local vers Azure Active Directory, créez une nouvelle règle de synchronisation entrante. Lancez **l’éditeur de règles de synchronisation**, sélectionnez la direction **Entrante**, puis sélectionnez **Ajouter une nouvelle règle**. 

 ![Capture d’écran montrant l’« éditeur de règles de synchronisation » avec le réglage « Entrante » et le bouton « Ajouter une nouvelle règle » sélectionnés.](media/how-to-connect-fix-default-rules/default3a.png)

Suivez votre propre convention d’affectation de noms pour nommer la règle. Ici, nous utilisons **Personnalisé Entrante à partir d’AD - utilisateur**. Cela signifie que la règle est une règle personnalisée, et une règle entrante à partir de l’espace de connecteur Active Directory vers le métaverse.   

 ![Créer une règle de synchronisation de trafic entrant](media/how-to-connect-fix-default-rules/default3b.png)

Fournissez votre propre description de la règle, afin que sa maintenance ultérieure soit aisé. Par exemple, la description peut être basée sur l’objectif de la règle et la raison pour laquelle elle est nécessaire.

Effectuez vos sélections pour les champs **Système connecté**, **Type d’objet système connecté**, et **Type d’objet métaverse**.

Spécifiez la valeur de priorité comprise entre 0 et 99 (plus le nombre est faible, plus la priorité est élevée). Pour les champs **Balise**, **Activer la synchronisation de mot de passe**, et **Désactivé**, utilisez les sélections par défaut.

Laissez le champ **Filtre d’étendue** vide. Cela signifie que la règle s’applique à tous les objets joints entre le système Active Directory connecté et le métaverse.

Laissez le champ **Règles de jointure** vide. Cela signifie que cette règle utilise la condition de jointure définie dans la règle par défaut standard. Il s’agit d’une autre raison pour ne pas désactiver ou supprimer la règle par défaut standard. S’il n’existe aucune condition de jointure, l’attribut ne circulera pas. 

Ajoutez les transformations appropriées pour votre attribut. Vous pouvez affecter une constante, pour qu’une valeur constante circule vers votre attribut cible. Vous pouvez utiliser un mappage direct entre l’attribut source ou cible. Ou bien, vous pouvez utiliser une expression pour l’attribut. Voici différentes [fonctions d’expression](./reference-connect-sync-functions-reference.md) que vous pouvez utiliser.

#### <a name="add-an-outbound-sync-rule"></a>Ajouter une règle de synchronisation sortante
Pour lier l’attribut au répertoire cible, vous devez créer une règle sortante. Cela signifie que la source est le métaverse et la cible est le système connecté. Pour créer une règle sortante, démarrez **l’éditeur de règles de synchronisation**, réglez **Direction** sur **Sortant**, puis sélectionnez **Ajouter une nouvelle règle**. 

![Éditeur de règles de synchronisation](media/how-to-connect-fix-default-rules/default3c.png)

Comme avec la règle de trafic entrant, vous pouvez utiliser votre propre convention d’affectation de noms pour la nommer. Sélectionnez le **système connecté** en tant que locataire Azure AD, puis sélectionnez l’objet de système connecté auquel vous souhaitez définir la valeur d’attribut. Définissez la priorité entre 0 et 99. 

![Créer une règle de synchronisation de trafic sortant](media/how-to-connect-fix-default-rules/default3d.png)

Laissez le **filtre d’étendue** et les **règles de jointure** vides. Renseignez la transformation comme étant constante, directe ou une expression. 

Vous savez maintenant comment rendre un nouvel attribut pour un flux d’objet utilisateur à partir d’Active Directory vers Azure Active Directory. Vous pouvez utiliser ces étapes pour mapper tout attribut provenant de n’importe quel objet à la source et à la cible. Pour plus d’informations, consultez [Création de règles de synchronisation personnalisées](how-to-connect-create-custom-sync-rule.md) et [Préparer pour configurer des utilisateurs](/office365/enterprise/prepare-for-directory-synchronization).

### <a name="override-the-value-of-an-existing-attribute"></a>Remplacer la valeur d’un attribut existant
Vous souhaiterez peut-être substituer la valeur d’un attribut qui a déjà été mappée. Par exemple, si vous souhaitez toujours définir une valeur null à un attribut dans Azure AD, créez simplement une règle entrante uniquement. Faites que la valeur constante, `AuthoritativeNull`, circule vers l’attribut cible. 

>[!NOTE] 
> Utilisez `AuthoritativeNull` au lieu de `Null` dans ce cas. Car la valeur non null remplace la valeur null, même si elle a une priorité inférieure (une valeur numérique plus élevée dans la règle). `AuthoritativeNull`, de l’autre côté, n’est pas remplacé par une valeur non null par d’autres règles. 

### <a name="dont-sync-existing-attribute"></a>Ne pas synchroniser un attribut existant
Si vous souhaitez exclure un attribut de la synchronisation, utilisez la fonctionnalité de filtrage d’attributs fournie dans Azure AD Connect. Lancez **Azure AD Connect** depuis l’icône du bureau, puis sélectionnez **Personnaliser les options de synchronisation**.

![Options des tâches supplémentaires Azure AD Connect](media/how-to-connect-fix-default-rules/default4.png)

 Assurez-vous que **le filtrage d’attributs et d’applications Azure AD** est sélectionné, puis sélectionnez **Suivant**.

![Fonctionnalités facultatives Azure AD Connect](media/how-to-connect-fix-default-rules/default5.png)

Effacer les attributs que vous souhaitez exclure de la synchronisation.

![Attributs Azure AD Connect](media/how-to-connect-fix-default-rules/default6a.png)

## <a name="change-scoping-filter"></a>Modifier un filtre d’étendue
Azure AD Sync s’occupe de la plupart des objets. Vous pouvez réduire l’étendue des objets et réduire le nombre d’objets à exporter, sans modifier les règles de synchronisation par défaut standards. 

Utilisez une des méthodes suivantes pour réduire l’étendue des objets que vous synchronisez :

- Attribut cloudFiltered
- Filtrage de l’unité d’organisation

Si vous réduisez l’étendue des utilisateurs synchronisés, la synchronisation du hachage de mot de passe s’arrête également pour les utilisateurs filtrés. Si les objets sont déjà synchronisés, après avoir réduit la portée, les objets filtrés sont supprimés du répertoire cible. Pour cette raison, assurez-vous de définir l’étendue très soigneusement.

>[!IMPORTANT] 
> L’augmentation de l’étendue des objets configuré par Azure AD Connect n’est pas recommandée. Cela rend difficile pour l’équipe de support Microsoft de comprendre les personnalisations. Si vous devez augmenter l’étendue des objets, modifiez la règle existante, clonez-la et désactivez la règle d’origine. 

### <a name="cloudfiltered-attribute"></a>Attribut cloudFiltered
Vous ne pouvez pas définir cet attribut dans Active Directory. Définissez la valeur de cet attribut en ajoutant une nouvelle règle de trafic entrant. Vous pouvez ensuite utiliser **Transformation** et **Expression** pour définir cet attribut dans le métaverse. L’exemple suivant montre que vous ne souhaitez pas synchroniser tous les utilisateurs dont le nom du service commence par **HRD** (non-respect de la casse) :

`cloudFiltered <= IIF(Left(LCase([department]), 3) = "hrd", True, NULL)`

Tout d’abord, nous avons converti le département à partir de la source (Active Directory) en minuscules. Ensuite, à l’aide de la fonction `Left`, nous avons pris uniquement les trois premiers caractères et les avons comparé avec `hrd`. Si cela correspond, la valeur est définie sur `True`, sinon `NULL`. Dans la définition de la valeur sur null, une autre règle avec une priorité inférieure (valeur numérique plus élevée) peut y écrire avec une condition différente. Lancez un aperçu sur un seul objet pour valider la règle de synchronisation, comme indiqué dans la section [Valider une règle de synchronisation](#validate-sync-rule).

![Options de création d’une règle de synchronisation de trafic entrant](media/how-to-connect-fix-default-rules/default7a.png)

### <a name="organizational-unit-filtering"></a>Filtrage de l’unité d’organisation
Vous pouvez créer une ou plusieurs unités d’organisation (UO) et déplacer les objets que vous ne souhaitez pas synchroniser vers ces unités. Ensuite, configurez le filtrage d’unité d’organisation dans Azure AD Connect. Lancez **Azure AD Connect** à partir de l’icône du bureau, puis sélectionnez les options suivantes. Vous pouvez également configurer le filtrage d’unité d’organisation au moment de l’installation d’Azure AD Connect. 

![Tâches supplémentaires Azure AD Connect](media/how-to-connect-fix-default-rules/default8.png)

Suivez l’assistant et désactivez les unités d’organisation que vous ne souhaitez pas synchroniser.

![Options de filtrage domaine et unité d’organisation Azure AD Connect](media/how-to-connect-fix-default-rules/default9.png)

## <a name="change-join-condition"></a>Modifier une condition de jointure
Utilisez les conditions de jointure par défaut configurées par Azure AD Connect. La modification des conditions de jointure par défaut rend difficile pour le support technique de Microsoft de comprendre les personnalisations et d’effectuer le support technique du produit.

## <a name="validate-sync-rule"></a>Valider une règle de synchronisation
Vous pouvez valider la règle de synchronisation qui vient d’être ajoutée à l’aide de la fonctionnalité d’aperçu, sans exécuter le cycle de synchronisation complète. Dans Azure AD Connect, sélectionnez **Service de synchronisation**.

![Azure AD Connect, avec le service de synchronisation mis en surbrillance](media/how-to-connect-fix-default-rules/default10.png)

Sélectionnez **Recherche de métaverse**. Sélectionnez l’objet d’étendue comme **personne**, sélectionnez **Ajouter une clause**et indiquez vos critères de recherche. Ensuite, sélectionnez **Recherche**, double-cliquez sur l’objet dans les résultats de recherche. Assurez-vous que vos données dans Azure AD Connect sont à jour pour cet objet, en exécutant l’importation et la synchronisation sur la forêt avant d’exécuter cette étape.

![Synchronization Service Manager](media/how-to-connect-fix-default-rules/default11.png)

Sur les **propriétés de l’objet métaverse**, sélectionnez **Connecteurs**, sélectionnez l’objet dans le connecteur correspondant (forêt), puis **Propriétés...** .

![Propriétés de l’objet métaverse](media/how-to-connect-fix-default-rules/default12.png)

Sélectionnez **Aperçu...**

![Propriétés de l’objet espace connecteur](media/how-to-connect-fix-default-rules/default13a.png)

Dans la fenêtre d’aperçu, sélectionnez **Générer un aperçu** et **Importer un flux d’attribut** dans le volet gauche.

![Capture d’écran montrant la fenêtre « Aperçu » avec la rubrique « Importer un flux d’attribut » et le bouton « Générer un aperçu » sélectionnés.](media/how-to-connect-fix-default-rules/default14.png)
 
Ici, notez que la règle nouvellement ajoutée est exécutée sur l’objet et qu’elle a défini l’attribut `cloudFiltered` sur True.

![PRÉVERSION](media/how-to-connect-fix-default-rules/default15a.png)
 
Pour comparer la règle modifiée avec la règle par défaut, exportez les deux règles séparément, en tant que fichiers texte. Ces règles sont exportées dans un fichier de script PowerShell. Vous pouvez les comparer à l’aide de n’importe quel outil de comparaison de fichier (par exemple, windiff) pour voir les modifications. 
 
Notez que dans la règle modifiée, l’attribut `msExchMailboxGuid` devient de type **Expression** au lieu de **Direct**. En outre, la valeur est modifiée possède les options **NULL** et **ExecuteOnce**. Vous pouvez ignorer les différences Identifié et Priorité. 

![sortie de l’outil Windiff](media/how-to-connect-fix-default-rules/default17.png)
 
Pour corriger vos règles et remettre les paramètres par défaut, supprimez la règle modifiée et activez la règle par défaut. Assurez-vous de ne pas perdre la personnalisation que vous tentez d’atteindre. Lorsque vous êtes prêt, exécutez la **synchronisation complète**.

## <a name="next-steps"></a>Étapes suivantes
- [Matériel et conditions préalables](how-to-connect-install-prerequisites.md) 
- [Paramètres Express](how-to-connect-install-express.md)
- [Paramètres personnalisés](how-to-connect-install-custom.md)