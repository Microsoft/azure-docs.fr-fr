---
title: Résoudre les problèmes de synchronisation d’un objet avec Azure Active Directory | Microsoft Docs
description: Résolvez les problèmes de synchronisation entre un objet et Azure Active Directory.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 08/10/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1a0c8a42edad08308095469039c048f8dd8552af
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94413460"
---
# <a name="troubleshoot-an-object-that-is-not-synchronizing-with-azure-active-directory"></a>Résoudre les problèmes de synchronisation d’un objet avec Azure Active Directory

Si un objet ne se synchronise pas comme prévu avec Microsoft Azure Active Directory (Azure AD), il peut y avoir plusieurs raisons à cela. Si vous avez reçu un e-mail d’erreur provenant d’Azure AD ou que vous voyez l’erreur dans Azure AD Connect Health, lisez plutôt [Résolution des erreurs lors de la synchronisation](tshoot-connect-sync-errors.md). En revanche, si vous essayez de résoudre le problème d’un objet qui ne se trouve pas dans Azure AD, cet article vous concerne. Il explique comment rechercher des erreurs dans la synchronisation Azure AD Connect des composants locaux.

>[!IMPORTANT]
>Dans le cas d’un déploiement Azure AD Connect version 1.1.749.0 ou ultérieure, utilisez la [tâche de résolution des problèmes](tshoot-connect-objectsync.md) disponible dans l’Assistant prévu pour résoudre les problèmes de synchronisation d’objets. 

## <a name="synchronization-process"></a>Processus de synchronisation

Avant d’examiner les problèmes de synchronisation, essayons de comprendre le processus de synchronisation d’Azure AD Connect :

  ![Schéma du processus de synchronisation d’Azure AD Connect](./media/tshoot-connect-object-not-syncing/syncingprocess.png)

### <a name="terminology"></a>**Terminologie**

* **CS :** Espace connecteur (table de base de données)
* **MV :** Métaverse (table de base de données)

### <a name="synchronization-steps"></a>**Étapes de la synchronisation**
Le processus de synchronisation comprend les étapes suivantes :

1. **Importation à partir d’AD :** les objets Active Directory sont placés dans l’espace Active Directory CS.

2. **Importation à partir d’Azure AD :** les objets Azure AD sont placés dans l’espace Azure AD CS.

3. **Synchronisation :** les règles de synchronisation entrante et les règles de synchronisation sortante sont exécutées dans l’ordre de priorité numérique, du plus petit au plus grand. Pour afficher les règles de synchronisation, vous pouvez accéder à l’éditeur de règles de synchronisation à partir des applications de bureau. Les règles de synchronisation entrante déplacent les données de CS dans MV. Les règles de synchronisation sortante déplacent les données de MV dans CS.

4. **Exportation vers AD :** À l’issue de la synchronisation, les objets sont exportés de l’espace Active Directory CS dans Active Directory.

5. **Exportation vers Azure AD :** À l’issue de la synchronisation, les objets sont exportés de l’espace Azure AD CS dans Azure AD.

## <a name="troubleshooting"></a>Dépannage

Pour rechercher les erreurs, regardez à différents endroits, dans l’ordre suivant :

1. Les [journaux d’activité des opérations](#operations) pour rechercher les erreurs identifiées par le moteur de synchronisation pendant l’importation et la synchronisation.
2. L’[espace connecteur](#connector-space-object-properties) pour rechercher les objets manquants et les erreurs de synchronisation.
3. Le [métaverse](#metaverse-object-properties) pour rechercher les problèmes liés aux données.

Démarrez [Synchronization Service Manager](how-to-connect-sync-service-manager-ui.md) avant de commencer ces étapes.

## <a name="operations"></a>Opérations
L’onglet **Operations** (Opérations) dans Synchronization Service Manager est l’endroit où vous devez commencer votre dépannage. Cet onglet renseigne sur les résultats des opérations les plus récentes. 

![Capture d’écran de Synchronization Service Manager, avec l’onglet Operations sélectionné](./media/tshoot-connect-object-not-syncing/operations.png)  

La partie supérieure de l’onglet **Operations** (Opérations) répertorie toutes les exécutions dans un ordre chronologique. Par défaut, le journal des opérations conserve les informations des sept derniers jours, mais vous pouvez modifier ce paramètre à l’aide du [planificateur](how-to-connect-sync-feature-scheduler.md). Recherchez toute exécution qui ne présente pas un état de **success** (réussite). Vous pouvez modifier le tri en cliquant sur les en-têtes.

La colonne **Status** (État) contient les informations les plus importantes et renseigne sur le problème le plus grave pour une exécution. Voici un récapitulatif rapide des états les plus courants par ordre de priorité d’inspection (sachant que * indique plusieurs chaînes d’erreur possibles).

| Statut | Commentaire |
| --- | --- |
| stopped- * |L’exécution n’a pas pu se terminer. Cela peut se produire, par exemple, si le système distant est arrêté et ne peut pas être contacté. |
| stopped-error-limit |Il existe plus de 5 000 erreurs. L’exécution a été automatiquement arrêtée en raison du grand nombre d’erreurs. |
| completed-\*-errors |L’exécution s’est terminée, mais il existe des erreurs (moins de 5 000) qui doivent être examinées. |
| completed-\*-warnings |L’exécution s’est terminée, mais des données n’ont pas l’état attendu. Si vous avez des erreurs, ce message n’est, en général, qu’un symptôme. N’explorez pas les avertissements avant d’avoir résolu les erreurs. |
| success |Aucun problème. |

Lorsque vous sélectionnez une ligne, la partie inférieure de l’onglet **Operations** (Opérations) est mise à jour pour afficher les détails de cette exécution. Situé tout à gauche dans cette zone, vous pouvez voir une liste intitulée **Step #** (Étape #). Cette liste ne s’affiche que si votre forêt contient plusieurs domaines et que chaque domaine est représenté par une étape. Le nom de domaine se trouve sous le titre **Partition**. Sous l’en-tête **(Synchronization Statistics)** (Statistiques de synchronisation), vous trouvez plus d’informations sur le nombre de modifications qui ont été traitées. Sélectionnez les liens pour obtenir la liste des objets modifiés. Si vous avez des objets comportant des erreurs, celles-ci s’affichent sous l’en-tête **Synchronization Errors** (Erreurs de synchronisation).

### <a name="errors-on-the-operations-tab"></a>Erreurs dans l’onglet Operations (Opérations)
Quand vous avez des erreurs, Synchronization Service Manager affiche l’objet comportant l’erreur et l’erreur elle-même sous forme de liens qui fournissent d’autres informations.

![Capture d’écran d’erreurs dans Synchronization Service Manager](./media/tshoot-connect-object-not-syncing/errorsync.png)  
Commencez par sélectionner la chaîne d’erreur. (Dans l’image précédente, la chaîne d’erreur est **sync-rule-error-function-triggered**.) Une présentation de l’objet s’affiche d’abord. Pour voir l’erreur détaillée, sélectionnez **Stack Trace** (Trace de la pile). Cette trace vous fournit des informations de débogage pour l’erreur.

Cliquez avec le bouton droit dans la zone **Call Stack Information** (Informations de la pile des appels), choisissez **Select All** (Tout sélectionner), puis **Copy** (Copier). Copiez ensuite la pile et examinez l’erreur dans l’éditeur de votre choix, par exemple le Bloc-notes.

Si l’erreur provient de **SyncRulesEngine** , les informations de la pile des appels affichent d’abord la liste de tous les attributs sur l’objet. Faites défiler vers le bas jusqu’à l’en-tête **InnerException =>** .  

  ![Capture d’écran de Synchronization Service Manager, répertoriant les informations d’erreur sous le titre InnerException =>](./media/tshoot-connect-object-not-syncing/errorinnerexception.png)
  
La ligne qui suit le titre renseigne sur l’erreur. Dans l’image précédente, l’erreur provient d’une règle de synchronisation personnalisée qui a été créée par Fabrikam.

Si l’erreur ne fournit pas suffisamment d’informations, il est temps d’examiner les données elles-mêmes. Sélectionnez le lien contenant l’identificateur d’objet et poursuivez la résolution des problèmes de [l’objet importé de l’espace connecteur](#cs-import).

## <a name="connector-space-object-properties"></a>Propriétés de l’objet espace connecteur
Si l’onglet [**Operations**](#operations) (Opérations) n’affiche aucune erreur, suivez l’objet espace connecteur depuis Active Directory jusqu’au métaverse et à Azure AD. Dans ce chemin d’accès, vous devez rechercher la cause du problème.

### <a name="searching-for-an-object-in-the-cs"></a>Recherche d’un objet dans l’espace connecteur

Dans Synchronization Service Manager, sélectionnez **Connectors** (Connecteurs), puis le connecteur Active Directory et enfin **Search Connector Space** (Rechercher dans l’espace connecteur).

Dans la zone **Scope** (Étendue), sélectionnez **RDN** lorsque vous souhaitez effectuer une recherche sur l’attribut CN, ou sélectionnez **DN or anchor** (Nom unique ou ancre) lorsque vous souhaitez effectuer une recherche sur l’attribut **distinguishedName**. Entrez une valeur et sélectionnez **Search** (Rechercher). 
 
![Capture d’écran d’une recherche dans l’espace connecteur](./media/tshoot-connect-object-not-syncing/cssearch.png)  

Si vous ne trouvez pas l’objet recherché, celui-ci peut avoir été filtré avec le [filtrage basé sur le domaine](how-to-connect-sync-configure-filtering.md#domain-based-filtering) ou le [filtrage basé sur l’unité d’organisation](how-to-connect-sync-configure-filtering.md#organizational-unitbased-filtering). Pour vérifier que le filtrage est bien configuré comme vous le souhaitez, lisez [Synchronisation Azure AD Connect : Configurer le filtrage](how-to-connect-sync-configure-filtering.md).

Vous pouvez effectuer une autre recherche utile en sélectionnant le connecteur Azure AD. Dans la zone **Scope** (Étendue), sélectionnez **Pending Import** (Importation en attente), puis cochez l’option **Add** (Ajouter). Cette recherche vous propose tous les objets synchronisés dans Azure AD qui ne peuvent pas être associés à un objet local.  

![Capture d’écran d’orphelins dans une recherche d’espace connecteur](./media/tshoot-connect-object-not-syncing/cssearchorphan.png) 
 
Ces objets ont été créés par un autre moteur de synchronisation, ou par un moteur de synchronisation doté d’une autre configuration de filtrage. Ces objets orphelins ne sont plus managés. Examinez cette liste et projetez de supprimer ces objets au moyen des applets de commande [Azure AD PowerShell](/previous-versions/azure/jj151815(v=azure.100)).

### <a name="cs-import"></a>Importation de CS
Quand vous ouvrez un objet CS, plusieurs onglets sont affichés en haut. L’onglet **Importer** affiche les données mises en lot après une importation.  

![Capture d’écran de la fenêtre des propriétés de l’objet espace connecteur, avec l’onglet Import sélectionné](./media/tshoot-connect-object-not-syncing/csobject.png)    

La colonne **Old Value** (Ancienne valeur) montre ce qui est actuellement stocké dans Connect, et la colonne **New Value** (Nouvelle valeur), ce qui a été reçu du système source et qui n’a pas encore été appliqué. Si une erreur est présente sur l’objet, les modifications ne sont pas traitées.

L’onglet **Synchronization Error** (Erreur de synchronisation) n’est visible dans la fenêtre **Connector Space Object Properties** (Propriétés de l’objet espace connecteur) que s’il existe un problème avec l’objet. Pour plus d’informations, consultez [Résoudre les erreurs de synchronisation dans l’onglet **Operations**](#errors-on-the-operations-tab).

![Capture d’écran de la fenêtre des propriétés de l’objet espace connecteur, avec l’onglet Synchronization Error sélectionné](./media/tshoot-connect-object-not-syncing/cssyncerror.png)  

### <a name="cs-lineage"></a>Lignage CS
L’onglet **Lineage** (Lignage) dans la fenêtre **Connector Space Object Properties** (Propriétés de l’objet espace connecteur) montre de quelle manière l’objet espace connecteur est lié à l’objet métaverse. Vous pouvez voir à quel moment le connecteur a importé pour la dernière fois une modification à partir du système connecté, et connaître les règles appliquées pour remplir les données dans le métaverse.  

![Capture d’écran de la fenêtre des propriétés de l’objet espace connecteur, avec l’onglet Lineage sélectionné](./media/tshoot-connect-object-not-syncing/cslineage.png)  

Dans l’image précédente, la colonne **Action** affiche une règle de synchronisation entrante avec l’action **Provision** (Provisionner). Cela signifie que tant que cet objet espace connecteur est présent, l’objet métaverse est conservé. Si la liste des règles de synchronisation affiche à la place une règle de synchronisation sortante avec une action **Provision** (Provisionner), cet objet est supprimé en même temps que l’objet métaverse.  

![Capture d’écran d’une zone de lignage dans l’onglet Lineage de la fenêtre des propriétés de l’objet espace connecteur](./media/tshoot-connect-object-not-syncing/cslineageout.png)  

Dans l’image précédente, vous pouvez également remarquer dans la colonne **PasswordSync** que l’espace connecteur entrant peut participer aux modifications du mot de passe, car une règle de synchronisation a la valeur **True**. Ce mot de passe est envoyé à Azure AD par l’intermédiaire de la règle sortante.

Sous l’onglet **Lineage** (Lignage), vous pouvez accéder au métaverse en sélectionnant [**Metaverse Object Properties**](#mv-attributes) (Propriétés de l’objet métaverse).

### <a name="preview"></a>PRÉVERSION
Le bouton **Preview** (Aperçu) se trouve en bas à gauche dans la fenêtre **Connector Space Object Properties** (Propriétés de l’objet espace connecteur). Sélectionnez ce bouton pour ouvrir la page **Preview** (Aperçu) dans laquelle vous pouvez synchroniser un objet. Cette page est utile si vous résolvez des problèmes liés aux règles de synchronisation personnalisée et que vous souhaitez voir l’effet d’une modification sur un seul objet. Vous pouvez sélectionner **Full sync** (Synchronisation complète) ou **Delta sync** (Synchronisation Delta). Vous pouvez également sélectionner **Generate Preview** (Générer l’aperçu), qui permet de conserver uniquement la modification en mémoire. Sinon, sélectionnez **Commit Preview** (Valider l’aperçu), qui met à jour le métaverse et implémente toutes les modifications dans les espaces connecteur cibles.  

![Capture d’écran de la page d’aperçu, avec l’option de démarrage de l’aperçu sélectionnée](./media/tshoot-connect-object-not-syncing/preview.png)  

Dans l’aperçu, vous pouvez inspecter l’objet et voir la règle appliquée pour un flux d’attribut particulier.  

![Capture d’écran de la page d’aperçu, montrant le flux des attributs d’importation](./media/tshoot-connect-object-not-syncing/previewresult.png)

### <a name="log"></a>Journal
À côté du bouton **Preview** (Aperçu), sélectionnez le bouton **Log** (Journal) pour ouvrir la page **Log** (Journal). Là, vous pouvez voir l’état de synchronisation et l’historique du mot de passe. Pour plus d’informations, consultez [Résoudre les problèmes de synchronisation du hachage de mot de passe avec Azure AD Connect Sync](tshoot-connect-password-hash-synchronization.md).

## <a name="metaverse-object-properties"></a>Propriétés de l’objet métaverse
Il est généralement préférable de commencer la recherche à partir de l’espace connecteur Active Directory source. Mais vous pouvez également démarrer la recherche depuis le métaverse.

### <a name="searching-for-an-object-in-the-mv"></a>Recherche d’un objet dans MV
Dans Synchronization Service Manager, sélectionnez **Metaverse Search** (Recherche dans le métaverse), comme illustré dans l’image suivante. Créez une requête capable de trouver l’utilisateur. Recherchez des attributs communs, comme **accountName** ( **sAMAccountName** ) et **userPrincipalName**. Pour plus d’informations, consultez [Recherche dans le métaverse de Sync Service Manager](how-to-connect-sync-service-manager-ui-mvsearch.md).

![Capture d’écran de Synchronization Service Manager, avec l’onglet Metaverse Search (Recherche dans le métaverse) sélectionné](./media/tshoot-connect-object-not-syncing/mvsearch.png)  

Dans la fenêtre **Résultats de la recherche** , cliquez sur l’objet.

Si vous ne trouvez pas l’objet, c’est qu’il n’a pas encore atteint le métaverse. Continuez à rechercher l’objet dans [l’espace de connecteur](#connector-space-object-properties) Active Directory. Si vous le trouvez dans l’espace connecteur Active Directory, il se peut qu’une erreur de synchronisation bloque l’entrée de l’objet dans le métaverse, ou qu’un filtre d’étendue de règles de synchronisation soit appliqué.

### <a name="object-not-found-in-the-mv"></a>Objet introuvable dans MV
Si l’objet se trouve dans l’espace Active Directory CS, mais pas dans le Métaverse, le filtre d’étendue est appliqué. Pour examiner le filtre d’étendue, accédez au menu de l’application de bureau, puis sélectionnez **Synchronization Rules Editor** (Éditeur de règles de synchronisation). Filtrez les règles applicables à l’objet en ajustant le filtre ci-dessous.

  ![Capture d’écran de l’éditeur de règles de synchronisation affichant une recherche de règles de synchronisation entrante](./media/tshoot-connect-object-not-syncing/syncrulessearch.png)

Affichez chaque règle de la liste ci-dessus et vérifiez le paramètre **Scoping filter (Filtre d’étendue)** . Dans le filtre d’étendue qui suit, si **isCriticalSystemObject** a la valeur Null ou FALSE, ou s’il n’est pas défini, il se situe dans l’étendue.

  ![Capture d’écran d’un filtre d’étendue dans une recherche de règles de synchronisation entrante](./media/tshoot-connect-object-not-syncing/scopingfilter.png)

Accédez à la liste d’attributs [CS Import](#cs-import) (Importation de CS), puis vérifiez quel filtre bloque l’objet et empêche son déplacement dans MV. La liste d’attributs **Connector Space** (Espace connecteur) affiche uniquement les attributs renseignés, ou ceux dont la valeur n’est pas de type Null. Par exemple, si l’attribut **isCriticalSystemObject** n’apparaît pas dans la liste, sa valeur est Null ou n’est pas renseignée.

### <a name="object-not-found-in-the-azure-ad-cs"></a>Objet introuvable dans l’espace Azure AD CS
Si l’objet est absent de l’espace connecteur d’Azure AD, mais qu’il existe dans le Métaverse, examinez le filtre d’étendue des règles de synchronisation sortante de l’espace connecteur correspondant, puis vérifiez si l’objet est filtré en raison des [attributs MV](#mv-attributes) qui ne répondent pas aux critères.

Pour afficher le filtre d’étendue sortant, sélectionnez les règles applicables de l’objet en ajustant le filtre ci-dessous. Affichez chaque règle et examinez la valeur correspondante de [l’attribut MV](#mv-attributes).

  ![Capture d’écran d’une recherche de règles de synchronisation sortante dans l’éditeur de règles de synchronisation](./media/tshoot-connect-object-not-syncing/outboundfilter.png)


### <a name="mv-attributes"></a>Attributs MV
Sous l’onglet **Attributes** (Attributs), vous pouvez voir les valeurs et quels connecteurs y ont contribué.  

![Capture d’écran de la fenêtre des propriétés de l’objet métaverse, avec l’onglet des attributs sélectionné](./media/tshoot-connect-object-not-syncing/mvobject.png)  

Si un objet n’est pas synchronisé, posez-vous les questions suivantes sur les états d’attribut dans le métaverse :
- L’attribut **cloudFiltered** est-il présent et défini sur **true**  ? Si c’est le cas, il a été filtré selon les étapes décrites dans [Filtrage par attribut](how-to-connect-sync-configure-filtering.md#attribute-based-filtering).
- L’attribut **sourceAnchor** est-il présent ? Si ce n’est pas le cas, utilisez-vous une topologie de forêt compte-ressource ? Si un objet est identifié en tant que boîte aux lettres liée (l’attribut **msExchRecipientTypeDetails** a la valeur **2** ), l’attribut **sourceAnchor** est fourni par la forêt avec un compte Active Directory activé. Assurez-vous que le compte principal a été importé et synchronisé correctement. Le compte principal doit être répertorié parmi les [connecteurs](#mv-connectors) pour l’objet.

### <a name="mv-connectors"></a>Connecteurs MV
L’onglet **Connectors** (Connecteurs) affiche tous les espaces connecteur qui ont une représentation de l’objet. 
 
![Capture d’écran de la fenêtre des propriétés de l’objet métaverse, avec l’onglet Connectors sélectionné](./media/tshoot-connect-object-not-syncing/mvconnectors.png)  

Vous devez disposer d’un connecteur pour :

- Chaque forêt Active Directory dans laquelle l’utilisateur est représenté. Cette représentation peut inclure des objets **Contact** et **foreignSecurityPrincipals**.
- Un connecteur dans Azure AD.

S’il vous manque le connecteur à Azure AD, consultez la section sur les [Attributs MV](#mv-attributes) pour vérifier les critères de provisionnement vers Azure AD.

Dans l’onglet **Connectors** (Connecteurs) , vous pouvez également accéder à l’ [objet espace connecteur](#connector-space-object-properties). Sélectionnez une ligne et cliquez sur **Propriétés**.

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur la [synchronisation Azure AD Connect](how-to-connect-sync-whatis.md).
- En savoir plus sur l’[identité hybride](whatis-hybrid-identity.md).