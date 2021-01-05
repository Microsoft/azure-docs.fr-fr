---
title: Approvisionnement des journaux dans le portail Azure Active Directory (version préliminaire) | Microsoft Docs
description: Présentation des journaux d’activité d’approvisionnement dans le portail Azure Active Directory
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 4b18127b-d1d0-4bdc-8f9c-6a4c991c5f75
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 10/07/2020
ms.author: markvi
ms.reviewer: arvinh
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2408db2d91740350405f11e2a1250ab9b3a4fe31
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96181201"
---
# <a name="provisioning-reports-in-the-azure-active-directory-portal-preview"></a>Approvisionnement des rapports dans le portail Azure Active Directory (version préliminaire)

L’architecture de création de rapports dans Azure Active Directory (Azure AD) comprend les composants suivants :

- **Activité** 
    - **Connexions** – Il s’agit d’informations sur l’utilisation des applications managées et les activités de connexion des utilisateurs.
    - **Journaux d’audit** - [Journaux d’audit](concept-audit-logs.md) – Fournit des informations sur les activités du système liées aux utilisateurs et à la gestion des groupes, les applications gérées et les activités de répertoire.
    - **Approvisionnement des journaux** : fournit une activité système sur les utilisateurs, les groupes et les rôles qui sont approvisionnés par le service d’approvisionnement Azure AD. 

- **Sécurité** 
    - **Connexions risquées** : une [connexion risquée](../identity-protection/overview-identity-protection.md) est une tentative de connexion susceptible de provenir d’un utilisateur autre que le propriétaire légitime d’un compte d’utilisateur.
    - **Utilisateurs avec indicateur de risque** : un [utilisateur à risque](../identity-protection/overview-identity-protection.md) correspond à un indicateur de compte d’utilisateur susceptible d’être compromis.

Cette rubrique présente une vue d’ensemble du rapport d’approvisionnement.

## <a name="prerequisites"></a>Prérequis

### <a name="who-can-access-the-data"></a>Qui peut accéder aux données ?
* Propriétaires d’applications (peuvent afficher les journaux des applications qu’ils possèdent)
* Utilisateurs des rôles Administrateur de la sécurité, Lecteur de sécurité, Lecteur de rapports, Administrateur d’application et Administrateur d’application cloud
* Les administrateurs généraux


### <a name="what-azure-ad-license-do-you-need-to-access-provisioning-activities"></a>De quelle licence Azure AD avez-vous besoin pour accéder aux activités d’approvisionnement ?

Votre client doit avoir une licence Azure AD Premium associée pour afficher tous les rapports d’activités d’approvisionnement. Consultez [Bien démarrer avec Azure Active Directory Premium](../fundamentals/active-directory-get-started-premium.md) pour mettre à niveau votre édition d’Azure Active Directory. 

## <a name="provisioning-logs"></a>Journaux d’approvisionnement

Les journaux d’approvisionnement fournissent des réponses aux questions suivantes :

* Quels groupes ont été créés avec succès dans ServiceNow ?
* Quels rôles ont été importés d’Amazon Web Services ?
* Quels utilisateurs n’ont pas été créés avec succès dans DropBox ?

Vous pouvez accéder aux journaux d’approvisionnement en sélectionnant **Provisionner des journaux** dans la section **Surveillance** du panneau **Azure Active Directory** dans le [Portail Azure](https://portal.azure.com). L’affichage de certains enregistrements d’approvisionnement dans le portail peut prendre jusqu’à deux heures.

![Journaux d’approvisionnement](./media/concept-provisioning-logs/access-provisioning-logs.png "Journaux d’approvisionnement")


Un journal d’approvisionnement comporte un affichage de liste par défaut qui indique :

- L’identité
- L’action
- Le système source
- Le système cible
- Le statut
- La date


![Colonnes par défaut](./media/concept-provisioning-logs/default-columns.png "Colonnes par défaut")

Vous pouvez personnaliser le mode Liste en cliquant sur **Colonnes** dans la barre d’outils.

![Sélecteur de colonnes](./media/concept-provisioning-logs/column-chooser.png "Sélecteur de colonnes")

Cela vous permet d’afficher des champs supplémentaires, ou de supprimer des champs qui sont déjà affichés.

![Colonnes disponibles](./media/concept-provisioning-logs/available-columns.png "Colonnes disponibles")

Sélectionnez un élément dans la vue sous forme de liste pour obtenir des informations plus détaillées.

![Informations détaillées](./media/concept-provisioning-logs/steps.png "Filtrer")


## <a name="filter-provisioning-activities"></a>Filtrer les activités d’approvisionnement

Vous pouvez filtrer vos données d'approvisionnement. Certaines valeurs de filtre sont renseignées dynamiquement en fonction de votre locataire. Si, par exemple, vous n’avez pas d’événements de création dans votre locataire, il n’y aura pas d’option de filtre à créer.
Dans l'affichage par défaut, vous pouvez sélectionner les filtres suivants :

- Identité
- Date
- Statut
- Action


![Ajouter des filtres](./media/concept-provisioning-logs/default-filter.png "Filtrer")

Le filtre **Identité** vous permet de spécifier le nom ou l’identité qui vous intéresse. Cette identité peut être un utilisateur, un groupe, un rôle ou un autre objet. Vous pouvez effectuer une recherche par nom ou ID de l’objet. L’ID varie selon le scénario. Par exemple, lors de l’approvisionnement d’un objet d’Azure AD vers SalesForce, l’ID source est l’ID d’objet de l’utilisateur dans Azure AD, tandis qu’IDCible est l’ID de l’utilisateur dans Salesforce. Lors de l’approvisionnement de Workday vers Active Directory, l’ID source est l’ID employé du travailleur Workday. Notez qu’il se peut que le nom de l’utilisateur ne soit pas toujours présent dans la colonne d’identité. Il y aura toujours un ID. 


Le filtre **Date** vous permet de définir un intervalle de temps pour les données renvoyées.  
Les valeurs possibles sont les suivantes :

- 1 mois
- 7 jours
- 30 jours
- 24 heures
- Intervalle de temps personnalisé

Lorsque vous sélectionnez un délai d’exécution personnalisé, vous pouvez configurer une date de début et une date de fin.


Le filtre **État** vous permet de sélectionner les états suivants :

- Tous
- Succès
- Échec
- Ignoré



Le filtre **Action** vous permet de filtrer les éléments suivants :

- Créer 
- Update
- DELETE
- Disable
- Autres

En plus des filtres de l'affichage par défaut, vous pouvez également définir les filtres suivants :

- ID de travail
- ID du cycle
- ID de la modification
- ID de la source
- ID de la cible
- Application


![Choisir un champ](./media/concept-provisioning-logs/add-filter.png "Choisir un champ")


- **ID de tâche** : un ID de tâche unique est associé à chaque application pour laquelle vous avez activé l’approvisionnement.   

- **ID de cycle** : identifie de façon unique le cycle d’approvisionnement. Vous pouvez partager cet ID avec le support pour rechercher le cycle dans lequel cet événement s’est produit.

- **Changer l’ID :** identificateur unique de l’événement d’approvisionnement. Vous pouvez partager cet ID avec le support pour rechercher l’événement d’approvisionnement.   


- **Système source** : vous permet de spécifier l'emplacement à partir duquel l'identité est approvisionnée. Par exemple, lors de l’approvisionnement d’un objet d’Azure AD vers ServiceNow, le système source est Azure AD. 

- **Système cible** : vous permet de spécifier l'emplacement où l'identité est approvisionnée. Par exemple, lors de l’approvisionnement d’un objet d’Azure AD vers ServiceNow, le système cible est ServiceNow. 

- **Application** : vous permet d'afficher uniquement les enregistrements des applications dont le nom d'affichage contient une chaîne spécifique.

 

## <a name="provisioning-details"></a>Détails de l’approvisionnement 

Lorsque vous sélectionnez un élément dans la vue de la liste d’approvisionnement, vous pouvez obtenir plus de détails sur cet élément.
Les détails sont regroupés en fonction des catégories suivantes :

- Étapes

- Résolution des problèmes et recommandations

- Propriétés modifiées

- Résumé


![Détails de l’approvisionnement](./media/concept-provisioning-logs/provisioning-tabs.png "Onglets")



### <a name="steps"></a>Étapes

L’onglet **Étapes** présente les étapes nécessaires pour approvisionner un objet. L’approvisionnement d’un objet peut être constitué de quatre étapes : 

- Importer un objet
- Déterminer si l’objet est dans la portée
- Faire correspondre l’objet entre la source et la cible
- Approvisionner l’objet (effectuer une action : il peut s’agir d’une opération de création, de mise à jour, de suppression ou de désactivation)



![La capture d’écran montre l’onglet Étapes, qui présente les étapes d’approvisionnement.](./media/concept-provisioning-logs/steps.png "Filtrer")


### <a name="troubleshoot-and-recommendations"></a>Résolution des problèmes et recommandations


L’onglet de **résolution des problèmes et recommandations** fournit le code d’erreur et la raison. Les informations sur l’erreur sont uniquement disponibles en cas de défaillance. 


### <a name="modified-properties"></a>Propriétés modifiées

Les **propriétés modifiées** affichent l’ancienne valeur et la nouvelle valeur. Dans les cas où il n’y a pas d’ancienne valeur, la colonne de l’ancienne valeur est vide. 


### <a name="summary"></a>Résumé

L’onglet **résumé** fournit une vue d’ensemble des événements et des identificateurs de l’objet dans les systèmes source et cible. 

## <a name="what-you-should-know"></a>Ce que vous devez savoir

- Le Portail Azure stocke les données de provisionnement rapportées pendant 30 jours si vous disposez d’une édition Premium et pendant 7 jours si vous disposez d’une édition gratuite. Les journaux de provisionnement peuvent être publiés dans [Log Analytics](../app-provisioning/application-provisioning-log-analytics.md) pour être conservés au-delà de 30 jours. 

- Vous pouvez utiliser l’attribut Changer l’ID comme identificateur unique. C’est, par exemple, utile lors de l’interaction avec le support technique.

- Il n’existe actuellement aucune option permettant de télécharger les données de configuration en tant que fichier CSV, mais vous pouvez exporter les données à l’aide de [Microsoft Graph](/graph/api/provisioningobjectsummary-list?tabs=http&view=graph-rest-beta).

- Vous pouvez voir des événements ignorés pour les utilisateurs qui ne sont pas dans l’étendue. Cela est prévu, en particulier lorsque l’étendue de synchronisation est définie sur tous les utilisateurs et groupes. Notre service évalue tous les objets du locataire, y compris ceux qui sont en dehors de l’étendue. 

- Les journaux d’approvisionnement ne sont actuellement pas disponibles dans le cloud Government. Si vous ne parvenez pas à accéder aux journaux d’approvisionnement, utilisez les journaux d’audit comme solution temporaire de contournement.  

## <a name="error-codes"></a>Codes d’erreur

Utilisez le tableau ci-dessous pour mieux comprendre comment résoudre les erreurs que vous pouvez trouver dans les journaux d’approvisionnement. Pour tous les codes d’erreur manquants, fournissez vos commentaires à l’aide du lien en bas de cette page. 

|Code d'erreur|Description|
|---|---|
|Conflict, EntryConflict|Corrigez les valeurs d’attribut en conflit dans Azure AD ou l’application, ou passez en revue votre configuration d’attribut correspondante si le compte d’utilisateur en conflit était supposé être mis en correspondance et pris en charge. Consultez la [documentation](../app-provisioning/customize-application-attributes.md) suivante pour plus d’informations sur la configuration des attributs correspondants.|
|TooManyRequests|L’application cible a rejeté cette tentative de mise à jour de l’utilisateur, car elle est surchargée et reçoit trop de requêtes. Il n’y a rien à faire. Cette tentative sera automatiquement supprimée. Microsoft a également été informé de ce problème.|
|InternalServerError |L’application cible a retourné une erreur inattendue. Il peut y avoir un problème de service avec l’application cible qui empêche cela de fonctionner. Cette tentative sera automatiquement mise hors service dans 40 minutes.|
|InsufficientRights, MethodNotAllowed, NotPermitted, Unauthorized| Azure AD a pu s’authentifier auprès de l’application cible, mais n’a pas été autorisé à effectuer la mise à jour. Passez en revue toutes les instructions fournies par l’application cible, ainsi que le [didacticiel](../saas-apps/tutorial-list.md) de l’application correspondante.|
|UnprocessableEntity|L’application cible a renvoyé une réponse inattendue. La configuration de l’application cible n’est peut-être pas correcte, ou il peut y avoir un problème de service avec l’application cible qui empêche cela de fonctionner.|
|WebExceptionProtocolError |Une erreur de protocole HTTP s’est produite lors de la connexion à l’application cible. Il n’y a rien à faire. Cette tentative sera automatiquement mise hors service dans 40 minutes.|
|InvalidAnchor|Un utilisateur qui a été précédemment créé ou mis en correspondance par le service d’approvisionnement n’existe plus. Vérifiez que l’utilisateur existe. Pour forcer une nouvelle correspondance de tous les utilisateurs, utilisez l’API MS Graph pour [redémarrer un travail](/graph/api/synchronization-synchronizationjob-restart?tabs=http&view=graph-rest-beta). Notez que le redémarrage de l’approvisionnement déclenche un cycle initial, ce qui peut prendre du temps. Il supprime également le cache utilisé par le service de provisionnement pour fonctionner, ce qui signifie que tous les utilisateurs et groupes du locataire devront être réévalués et que certains événements d’approvisionnement pourraient être supprimés.|
|NotImplemented | L’application cible a retourné une réponse inattendue. La configuration de l’application n’est peut-être pas correcte, ou il peut y avoir un problème de service avec l’application cible qui empêche cela de fonctionner. Passez en revue toutes les instructions fournies par l’application cible, ainsi que le [didacticiel](../saas-apps/tutorial-list.md) de l’application correspondante. |
|MandatoryFieldsMissing, MissingValues |L’utilisateur n’a pas pu être créé, car des valeurs requises sont manquantes. Corrigez les valeurs d’attribut manquantes dans l’enregistrement source ou vérifiez la configuration de l’attribut correspondant pour vous assurer que les champs obligatoires ne sont pas omis. [En savoir plus](../app-provisioning/customize-application-attributes.md) sur la configuration des attributs correspondants.|
|SchemaAttributeNotFound |Impossible d’effectuer l’opération, car un attribut spécifié n’existe pas dans l’application cible. Consultez la [documentation](../app-provisioning/customize-application-attributes.md) sur la personnalisation des attributs et assurez-vous que votre configuration est correcte.|
|InternalError |Une erreur de service interne s’est produite au sein du service de provisionnement Azure AD. Il n’y a rien à faire. Cette tentative sera automatiquement mise hors service dans 40 minutes.|
|InvalidDomain |L’opération n’a pas pu être effectuée en raison d’une valeur d’attribut contenant un nom de domaine non valide. Mettez à jour le nom de domaine sur l’utilisateur ou ajoutez-le à la liste autorisée dans l’application cible. |
|Délai d'expiration |L’opération n’a pas pu aboutir car l’application cible a mis trop de temps à répondre. Il n’y a rien à faire. Cette tentative sera automatiquement mise hors service dans 40 minutes.|
|LicenseLimitExceeded|L’utilisateur n’a pas pu être créé dans l’application cible, car il n’existe aucune licence disponible pour cet utilisateur. Procurez-vous des licences supplémentaires pour l’application cible, ou passez en revue vos attributions d’utilisateurs et la configuration de mappage des attributs pour vous assurer que les utilisateurs appropriés sont affectés avec les attributs appropriés.|
|DuplicateTargetEntries  |L’opération n’a pas pu aboutir car plusieurs utilisateurs de l’application cible ont été trouvés avec les attributs correspondants configurés. Supprimez l’utilisateur en double de l’application cible ou reconfigurez vos mappages d’attributs comme décrit [ici](../app-provisioning/customize-application-attributes.md).|
|DuplicateSourceEntries | L’opération n’a pas pu aboutir car plusieurs utilisateurs ont été trouvés avec les attributs correspondants configurés. Supprimez l’utilisateur en double ou reconfigurez vos mappages d’attributs comme décrit [ici](../app-provisioning/customize-application-attributes.md).|
|ImportSkipped | Lors de l'évaluation de chaque utilisateur, nous essayons d'importer celui-ci à partir du système source. Cette erreur se produit généralement lorsque la propriété correspondante n'a pas été attribuée à l'utilisateur importé dans vos mappages d'attributs. En l'absence de valeur sur l'objet utilisateur pour l'attribut correspondant, nous ne pouvons pas évaluer l'étendue et la correspondance, ni exporter les modifications. Remarque : la présence de cette erreur n'indique pas que l'utilisateur se trouve dans l'étendue car nous n'avons pas encore évalué l'étendue de l'utilisateur.|
|EntrySynchronizationSkipped | Le service d'approvisionnement a interrogé le système source et identifié l'utilisateur. Aucune autre mesure n'a été prise à l'égard de l'utilisateur et il a été ignoré. L'omission peut être due au fait que l'utilisateur est hors étendue, ou qu'il figure déjà dans le système cible et qu'aucune autre modification n'est nécessaire.|
|SystemForCrossDomainIdentityManagementMultipleEntriesInResponse| Lors de l’exécution d’une requête GET pour récupérer un utilisateur ou un groupe, nous avons reçu plusieurs utilisateurs ou groupes dans la réponse. Nous nous attendions à ne recevoir qu’un seul utilisateur ou groupe dans la réponse. Si, [par exemple](../app-provisioning/use-scim-to-provision-users-and-groups.md#get-group), nous effectuons une requête GET pour récupérer un groupe et fournir un filtre permettant d’exclure des membres et que votre point de terminaison SCIM retourne les membres, nous lèverons cette erreur.|

## <a name="next-steps"></a>Étapes suivantes

* [Vérifier l’état de l’approvisionnement d’utilisateurs](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md)
* [Problèmes lors de la configuration de l’approvisionnement des utilisateurs pour une application relevant de la galerie Azure AD](../app-provisioning/application-provisioning-config-problem.md)
* [Provisionnement de journaux de l’API Graph](/graph/api/resources/provisioningobjectsummary?view=graph-rest-beta)