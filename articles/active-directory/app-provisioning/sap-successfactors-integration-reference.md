---
title: Référence sur l’intégration d’Azure AD et de SAP SuccessFactors
description: Présentation technique approfondie de l’approvisionnement piloté par les RH de SAP SuccessFactors
services: active-directory
author: cmmdesai
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: reference
ms.workload: identity
ms.date: 07/20/2020
ms.author: chmutali
ms.openlocfilehash: 805cdc0713afd43502bb224cce60167adbc418ee
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90969517"
---
# <a name="how-azure-active-directory-provisioning-integrates-with-sap-successfactors"></a>Intégration de l’approvisionnement Azure Active Directory avec SAP SuccessFactors 

Le [service d’approvisionnement d’utilisateur Azure Active Directory](../app-provisioning/user-provisioning.md) s’intègre à [SuccessFactors Central Employee](https://www.successfactors.com/products-services/core-hr-payroll/employee-central.html) afin de gérer le cycle de vie des identités des utilisateurs. Azure Active Directory propose trois intégrations prédéfinies : 

* [Provisionnement d’utilisateurs SuccessFactors vers Active Directory local](../saas-apps/sap-successfactors-inbound-provisioning-tutorial.md)
* [Provisionnement d’utilisateurs SuccessFactors vers Azure Active Directory](../saas-apps/sap-successfactors-inbound-provisioning-cloud-only-tutorial.md)
* [SuccessFactors Writeback](../saas-apps/sap-successfactors-writeback-tutorial.md)

Cet article explique le fonctionnement de l’intégration et comment vous pouvez personnaliser le comportement d’approvisionnement pour différents scénarios de RH. 

## <a name="establishing-connectivity"></a>Établir une connectivité 
Le service d’approvisionnement Azure AD utilise l’authentification de base pour se connecter aux points de terminaison de l’API OData centrale de l’employé. Quand vous configurez l’application de configuration SuccessFactors, utilisez le paramètre *URL du locataire* dans la section *Informations d’identification de l’administrateur* pour configurer l’[URL du centre de données de l’API](https://apps.support.sap.com/sap/support/knowledge/en/2215682). 

Pour renforcer la connectivité entre le service de provisionnement Azure AD et SuccessFactors, vous pouvez ajouter les plages d’adresses IP Azure AD dans la liste verte d’IP SuccessFactors à l’aide des étapes décrites ci-dessous :

1. Téléchargez les [dernières plages IP](https://www.microsoft.com/download/details.aspx?id=56519) pour le cloud public Azure 
1. Ouvrez le fichier et recherchez la balise **AzureActiveDirectory** 

   >[!div class="mx-imgBorder"] 
   >![Plage d’adresses IP Azure AD](media/sap-successfactors-integration-reference/azure-active-directory-ip-range.png)

1. Copiez toutes les plages d’adresses IP répertoriées dans l’élément *addressPrefixes* et utilisez la plage pour créer votre liste de restrictions d’adresses IP.
1. Traduisez les valeurs CIDR en plages d’adresses IP.  
1. Connectez-vous au portail d’administration SuccessFactors pour ajouter des plages d’adresses IP à la liste verte. Voir la [note de support SAP n° 2253200](https://apps.support.sap.com/sap/support/knowledge/en/2253200) Vous pouvez maintenant [entrer des plages d’adresses IP](https://answers.sap.com/questions/12882263/whitelisting-sap-cloud-platform-ip-address-range-i.html) dans cet outil. 

## <a name="supported-entities"></a>Entités prises en charge
Pour chaque utilisateur de SuccessFactors, le service d’approvisionnement Azure AD récupère les entités suivantes. Chaque entité est développée à l’aide du paramètre de requête de l’API OData *$expand*. Reportez-vous à la colonne *Règle de récupération* ci-dessous. Certaines entités sont développées par défaut, tandis que certaines entités sont développées uniquement si un attribut spécifique est présent dans le mappage. 

| \# | Entité SuccessFactors                  | Nœud OData     | Règle de récupération |
|----|----------------------------------------|------------------------------|------------------|
| 1  | PerPerson                              | *nœud racine*                  | Toujours           |
| 2  | PerPersonal                            | personalInfoNav              | Toujours           |
| 3  | PerPhone                               | phoneNav                     | Toujours           |
| 4  | PerEmail                               | emailNav                     | Toujours           |
| 5  | EmpEmployment                          | employmentNav                | Toujours           |
| 6  | Utilisateur                                   | employmentNav/userNav        | Toujours           |
| 7  | EmpJob                                 | employmentNav/jobInfoNav     | Toujours           |
| 8  | EmpEmploymentTermination               | activeEmploymentsCount       | Toujours           |
| 9  | FOCompany                              | employmentNav/jobInfoNav/companyNav | Uniquement si l’attribut `company` ou `companyId` est mappé |
| 10 | FODepartment                           | employmentNav/jobInfoNav/departmentNav | Uniquement si l’attribut `department` ou `departmentId` est mappé |
| 11 | FOBusinessUnit                         | employmentNav/jobInfoNav/businessUnitNav | Uniquement si l’attribut `businessUnit` ou `businessUnitId` est mappé |
| 12 | FOCostCenter                           | employmentNav/jobInfoNav/costCenterNav | Uniquement si l’attribut `costCenter` ou `costCenterId` est mappé |
| 13 | FODivision                             | employmentNav/jobInfoNav/divisionNav  | Uniquement si l’attribut `division` ou `divisionId` est mappé |
| 14 | FOJobCode                              | employmentNav/jobInfoNav/jobCodeNav  | Uniquement si l’attribut `jobCode` ou `jobCodeId` est mappé |
| 15 | FOPayGrade                             | employmentNav/jobInfoNav/payGradeNav  | Uniquement si l’attribut `payGrade` est mappé |
| 16 | FOLocation                             | employmentNav/jobInfoNav/locationNav  | Uniquement si l’attribut `location` est mappé |
| 17 | FOCorporateAddressDEFLT                | employmentNav/jobInfoNav/addressNavDEFLT  | Si le mappage contient l’un des attributs suivants : `officeLocationAddress,  officeLocationCity, officeLocationZipCode` |
| 18 | FOEventReason                          | employmentNav/jobInfoNav/eventReasonNav  | Uniquement si l’attribut `eventReason` est mappé |
| 19 | EmpGlobalAssignment                    | employmentNav/empGlobalAssignmentNav | Uniquement si `assignmentType` est mappé |
| 20 | EmploymentType Picklist                | employmentNav/jobInfoNav/employmentTypeNav | Uniquement si `employmentType` est mappé |
| 21 | EmployeeClass Picklist                 | employmentNav/jobInfoNav/employeeClassNav | Uniquement si `employeeClass` est mappé |
| 22 | Liste déroulante EmplStatus                    | employmentNav/jobInfoNav/emplStatusNav | Uniquement si `emplStatus` est mappé |
| 23 | Liste déroulante AssignmentType                | employmentNav/empGlobalAssignmentNav/assignmentTypeNav | Uniquement si `assignmentType` est mappé |

## <a name="how-full-sync-works"></a>Fonctionnement de la synchronisation complète
En fonction du mappage d’attributs, lors de la synchronisation complète du service d’approvisionnement Azure AD envoie la requête d’API OData « GET » suivante pour extraire les données effectives de tous les utilisateurs actifs. 

> [!div class="mx-tdCol2BreakAll"]
>| Paramètre | Description |
>| ----------|-------------|
>| Hôte de l’API OData | Ajoute https à *l’URL du locataire*. Exemple : `https://api4.successfactors.com` |
>| Point de terminaison d’API OData | `/odata/v2/PerPerson` |
>| Paramètre de requête OData $format | `json` |
>| Paramètre de requête OData $filter | `(personEmpTerminationInfoNav/activeEmploymentsCount ge 1) and (lastModifiedDateTime le <CurrentExecutionTime>)` |
>| Paramètre de requête OData $expand | La valeur de ce paramètre dépend des attributs mappés. Exemple : `employmentNav/userNav,employmentNav/jobInfoNav,personalInfoNav,personEmpTerminationInfoNav,phoneNav,emailNav,employmentNav/jobInfoNav/companyNav/countryOfRegistrationNav,employmentNav/jobInfoNav/divisionNav,employmentNav/jobInfoNav/departmentNav` |
>| Paramètre de requête OData customPageSize | `100` |

> [!NOTE]
> Au cours de la première synchronisation initiale complète, le service d’approvisionnement Azure AD n’extrait pas les données du worker inactives/terminées.

Pour chaque utilisateur SuccessFactors, le service d’approvisionnement recherche un compte dans la cible (Azure AD/local Active Directory) à l’aide de l’attribut correspondant défini dans le mappage. Par exemple : si *personIdExternal* est mappé à *employeeId* et est défini en tant qu’attribut correspondant, alors le service d’approvisionnement utilise la valeur *personIdExternal* pour rechercher l’utilisateur avec *filtre employeeId*. Si un utilisateur a trouvé une correspondance, il met à jour les attributs cibles. Si aucune correspondance n’est trouvée, elle crée une entrée dans la cible. 

Pour valider les données retournées par le point de terminaison de votre API OData pour une `personIdExternal`spécifique, mettez à jour les `SuccessFactorsAPIEndpoint` dans la requête d’API ci-dessous avec votre URL de serveur de centre de données d’API et utilisez un outil tel que [Postman](https://www.postman.com/downloads/) pour appeler la requête. 

```
https://[SuccessFactorsAPIEndpoint]/odata/v2/PerPerson?$format=json&
$filter=(personIdExternal in '[personIdExternalValue]')&
$expand=employmentNav/userNav,employmentNav/jobInfoNav,personalInfoNav,personEmpTerminationInfoNav,
phoneNav,phoneNav/phoneTypeNav,emailNav,employmentNav/jobInfoNav/businessUnitNav,employmentNav/jobInfoNav/companyNav,
employmentNav/jobInfoNav/companyNav/countryOfRegistrationNav,employmentNav/jobInfoNav/costCenterNav,
employmentNav/jobInfoNav/divisionNav,employmentNav/jobInfoNav/departmentNav,employmentNav/jobInfoNav/jobCodeNav,
employmentNav/jobInfoNav/locationNav,employmentNav/jobInfoNav/locationNav/addressNavDEFLT,employmentNav/jobInfoNav/payGradeNav,
employmentNav/empGlobalAssignmentNav,employmentNav/empGlobalAssignmentNav/assignmentTypeNav,employmentNav/jobInfoNav/emplStatusNav,
employmentNav/jobInfoNav/employmentTypeNav,employmentNav/jobInfoNav/employeeClassNav,employmentNav/jobInfoNav/eventReasonNav
```

## <a name="how-incremental-sync-works"></a>Synchronisation incrémentielle

Après une synchronisation complète, le service d’approvisionnement Azure AD gère `LastExecutionTimestamp` et l’utilise pour créer des requêtes delta pour la récupération des modifications incrémentielles. Les attributs d’horodatage présents dans chaque entité SuccessFactors, comme `lastModifiedDateTime`, `startDate`, `endDate`et `latestTerminationDate`, sont évalués pour déterminer si la modification se situe entre le `LastExecutionTimestamp` et le `CurrentExecutionTime`. Si c’est le cas, le changement d’entrée est considéré comme effectif et traité pour la synchronisation. 

## <a name="reading-attribute-data"></a>Lecture des données d'attribut

Quand le service d’approvisionnement Azure AD interroge SuccessFactors, il récupère un jeu de résultats JSON. Le jeu de résultats JSON comprend un certain nombre d’attributs stockés dans le Centre des employés. Par défaut, le schéma de configuration est configuré pour récupérer uniquement un sous-ensemble de ces attributs. 

Pour récupérer des attributs supplémentaires, suivez les étapes ci-dessous :
    
1. Accédez à la  -> **page de mappage des attributs** **Applications d’entreprise** -> **Application SuccessFactors** -> **Approvisionner** -> **Modifier l’approvisionnement**.
1. Faites défiler vers le bas et cliquez sur **Afficher les options avancées .** .
1. Cliquez sur **Modifier la liste des attributs de SuccessFactors**. 

   > [!NOTE] 
   > Si l’option **Modifier la liste des attributs de SuccessFactors** ne s’affiche pas dans le portail Azure, utilisez l’URL *https://portal.azure.com/?Microsoft_AAD_IAM_forceSchemaEditorEnabled=true* pour accéder à la page. 

1. La colonne **Expression d’API** de cette vue affiche les expressions de chemin JSON utilisées par le connecteur.

   >[!div class="mx-imgBorder"] 
   >![Expression d’API](media/sap-successfactors-integration-reference/jsonpath-api-expressions.png#lightbox)  

1. Vous pouvez modifier une valeur JSONPath existante ou ajouter un nouvel attribut avec une expression JSONPath valide au schéma. 

La section suivante fournit une liste de scénarios courants pour la modification des valeurs de JSONPath. 

## <a name="handling-different-hr-scenarios"></a>Gestion de différents scénarios RH

JSONPath est un langage de requête pour JSON qui est similaire à XPath pour XML. Comme XPath, JSONPath permet l’extraction et la filtration de données à partir de la charge utile JSON.

À l’aide de la transformation JSONPath, vous pouvez personnaliser le comportement de l’application de configuration Azure AD pour récupérer des attributs personnalisés et gérer les scénarios tels que la réembauche, la conversion de worker et l’affectation globale. 

Cette section décrit comment vous pouvez personnaliser l’application d’approvisionnement pour les scénarios RH suivants : 
* [Pour récupérer des attributs supplémentaires](#retrieving-additional-attributes)
* [Pour récupérer des attributs personnalisés](#retrieving-custom-attributes)
* [Gestion du scénario de conversion de worker](#handling-worker-conversion-scenario)
* [Gestion du scénario de réembauche](#handling-rehire-scenario)
* [Gestion du scénario d’attribution globale](#handling-global-assignment-scenario)
* [Gestion du scénario de tâches simultanées](#handling-concurrent-jobs-scenario)

### <a name="retrieving-additional-attributes"></a>Pour récupérer des attributs supplémentaires

La valeur par défaut du  schéma de l’application de configuration Azure AD SuccessFactors est fournie avec [plus de 90 attributs prédéfinis](sap-successfactors-attribute-reference.md). Pour ajouter d’autres attributs SuccessFactors au schéma d’approvisionnement, utilisez les étapes ci-dessous : 

1. Utilisez la requête OData ci-dessous pour récupérer les données d’un utilisateur de test valide du Centre des employés 

   ```
    https://[SuccessFactorsAPIEndpoint]/odata/v2/PerPerson?$format=json&
    $filter=(personIdExternal in '[personIdExternalValue]')&
    $expand=employmentNav/userNav,employmentNav/jobInfoNav,personalInfoNav,personEmpTerminationInfoNav,
    phoneNav,phoneNav/phoneTypeNav,emailNav,employmentNav/jobInfoNav/businessUnitNav,employmentNav/jobInfoNav/companyNav,
    employmentNav/jobInfoNav/companyNav/countryOfRegistrationNav,employmentNav/jobInfoNav/costCenterNav,
    employmentNav/jobInfoNav/divisionNav,employmentNav/jobInfoNav/departmentNav,employmentNav/jobInfoNav/jobCodeNav,
    employmentNav/jobInfoNav/locationNav,employmentNav/jobInfoNav/locationNav/addressNavDEFLT,employmentNav/jobInfoNav/payGradeNav,
    employmentNav/empGlobalAssignmentNav,employmentNav/empGlobalAssignmentNav/assignmentTypeNav,employmentNav/jobInfoNav/emplStatusNav,
    employmentNav/jobInfoNav/employmentTypeNav,employmentNav/jobInfoNav/employeeClassNav,employmentNav/jobInfoNav/eventReasonNav
   ```

1. Déterminer l’entité Centre des employés associée à l’attribut
   * Si l’attribut fait partie de l’entité *EmpEmployment*, recherchez l’attribut sous le nœud*employmentNav*. 
   * Si l’attribut fait partie de l’entité *Utilisateur*, recherchez l’attribut sous le nœud*employmentNav/userNav*.
   * Si l’attribut fait partie de l’entité *EmpJob*, recherchez l’attribut sous le nœud*employmentNav/jobInfoNav*. 
1. Construisez le chemin JSON associé à l’attribut et ajoutez ce nouvel attribut à la liste des attributs SuccessFactors. 
   * Exemple 1 : Supposons que vous souhaitez ajouter l’attribut *okToRehire*, qui fait partie de l’entité *employmentNav*, puis utiliser JSONPath  `$.employmentNav.results[0].okToRehire`
   * Exemple 2 : Supposons que vous souhaitez ajouter l’attribut *timeZone*, qui fait partie de l’entité *userNav*, puis utiliser JSONPath `$.employmentNav.results[0].userNav.timeZone`
   * Exemple 3 : Supposons que vous souhaitez ajouter l’attribut *flsaStatus*, qui fait partie de l’entité *jobInfoNav*, puis utiliser JSONPath `$.employmentNav.results[0].jobInfoNav.results[0].flsaStatus`
1. Enregistrez le schéma. 
1. Redémarrer l’approvisionnement.

### <a name="retrieving-custom-attributes"></a>Récupération des attributs personnalisés

Par défaut, les attributs personnalisés suivants sont prédéfinis dans l’application d’approvisionnement Azure AD SuccessFactors : 
* *custom01-CUSTOM15* de l’entité utilisateur (userNav)
* *customString1-customString15* de l’entité EmpEmployment (employmentNav) appelée *empNavCustomString1-empNavCustomString15*
* *customString1-customString15* de l’entité EmpEmployment (employmentNav) appelée *empJobNavCustomString1-empNavJobCustomString15*

Supposons que, dans votre instance d’employé centrale, l’*attribut customString35* dans *EmpJobInfo* stocke la description de l’emplacement. Vous souhaitez transmettre cette valeur à l’attribut *physicalDeliveryOfficeName*Active Directory. Pour configurer le mappage d’attributs pour ce scénario, utilisez les étapes indiquées ci-dessous : 

1. Modifiez la liste des attributs SuccessFactors pour ajouter un nouvel attribut appelé *empJobNavCustomString35*.
1. Définissez l’expression de l’API JSONPath pour cet attribut comme suit : `$.employmentNav.results[0].jobInfoNav.results[0].customString35`
1. Enregistrez et rechargez la modification de mappage dans le Portail Azure.  
1. Dans le panneau mappage d’attributs, mappez *empJobNavCustomString35* à *physicalDeliveryOfficeName*.
1. Sauvegardez le mappage.

Extension du scénario : 
* Si vous souhaitez mapper l’attribut *custom35* à partir de l’entité *Utilisateur*, utilisez JSONPath `$.employmentNav.results[0].userNav.custom35`
* Si vous souhaitez mapper l’attribut *customString35* à partir de l’entité *EmpEmployment*, utilisez JSONPath `$.employmentNav.results[0].customString35`

### <a name="handling-worker-conversion-scenario"></a>Gestion du scénario de conversion de worker

La conversion de worker est le processus qui consiste à convertir un employé à temps plein existant en un entrepreneur ou un entrepreneur en temps plein. Dans ce scénario, le Centre des employés ajoute une nouvelle entité *EmpEmployment* avec une nouvelle entité *Utilisateur* pour la même entité *Personne*. L’entité *Utilisateur*imbriquée sous l’entité *EmpEmployment* précédente est définie sur null. Pour gérer ce scénario afin que les nouvelles données d’emploi s’affichent lors d’une conversion, vous pouvez mettre à jour le schéma d’application d’approvisionnement en bloc à l’aide des étapes ci-dessous :  

1. Ouvrez le panneau mappage d’attributs de votre application d’approvisionnement de SuccessFactors. 
1. Faites défiler vers le bas et cliquez sur **Afficher les options avancées .** .
1. Cliquez sur le lien **Passez en revue votre schéma ici** pour ouvrir l’éditeur de schéma. 

   >![Capture d’écran montrant le lien Examiner votre schéma ici qui ouvre l’éditeur de schéma.](media/sap-successfactors-integration-reference/review-schema.png#lightbox)

1. Cliquez sur le lien **Télécharger** pour enregistrer une copie du schéma avant de le modifier. 

   >![Capture d’écran montrant l’Éditeur de schéma avec l’option Télécharger sélectionnée pour enregistrer une copie du schéma.](media/sap-successfactors-integration-reference/download-schema.png#lightbox)
1. Dans l’éditeur de schéma, appuyez sur Ctrl-H pour ouvrir le contrôle rechercher/remplacer.
1. Dans la zone de texte Rechercher, copiez et collez la valeur `$.employmentNav.results[0]`
1. Dans la zone de texte Remplacer, copiez et collez la valeur `$.employmentNav.results[?(@.userNav != null)]`. Notez l’espace blanc qui entoure l’opérateur `!=`, ce qui est important pour réussir le traitement de l’expression JSONPath. 
   >![find-replace-conversion](media/sap-successfactors-integration-reference/find-replace-conversion-scenario.png#lightbox)
1. Cliquez sur l’option « remplacer tout » pour mettre à jour le schéma. 
1. Enregistrez le schéma. 
1. Le processus ci-dessus met à jour toutes les expressions JSONPath comme suit : 
   * Ancien JSONPath : `$.employmentNav.results[0].jobInfoNav.results[0].departmentNav.name_localized`
   * Nouveau JSONPath : `$.employmentNav.results[?(@.userNav != null)].jobInfoNav.results[0].departmentNav.name_localized`
1. Redémarrer l’approvisionnement. 

### <a name="handling-rehire-scenario"></a>Gestion du scénario de réembauche

Il existe généralement deux options pour traiter les réembauches : 
* Option 1 : Créer un profil de personne dans le Centre des employés
* Option n°2 : Réutiliser un profil de personne existant dans le Centre des employés

Si votre processus RH utilise l’option 1, aucune modification n’est requise pour le schéma de configuration. Dans ce scénario 2, le Centre des employés ajoute une nouvelle entité *EmpEmployment* avec une nouvelle entité *Utilisateur* pour la même entité *Personne*. Contrairement au scénario de conversion, l'entité *Utilisateur* de l’entité précédente *EmpEmployment* n’a pas la valeur null. 

Pour gérer ce scénario de réembauche (option 2) afin que les dernières données d’emploi s’affichent pour les profils de réembauche, vous pouvez mettre à jour le schéma d’application d’approvisionnement en bloc à l’aide des étapes ci-dessous :  

1. Ouvrez le panneau mappage d’attributs de votre application d’approvisionnement de SuccessFactors. 
1. Faites défiler vers le bas et cliquez sur **Afficher les options avancées .** .
1. Cliquez sur le lien **Passez en revue votre schéma ici** pour ouvrir l’éditeur de schéma.   
1. Cliquez sur le lien **Télécharger** pour enregistrer une copie du schéma avant de le modifier.   
1. Dans l’éditeur de schéma, appuyez sur Ctrl-H pour ouvrir le contrôle rechercher/remplacer.
1. Dans la zone de texte Rechercher, copiez et collez la valeur `$.employmentNav.results[0]`
1. Dans la zone de texte Remplacer, copiez et collez la valeur `$.employmentNav.results[-1:]` Cette expression JSONPath retourne le dernier enregistrement *EmpEmployment*.   
1. Cliquez sur l’option « remplacer tout » pour mettre à jour le schéma. 
1. Enregistrez le schéma. 
1. Le processus ci-dessus met à jour toutes les expressions JSONPath comme suit : 
   * Ancien JSONPath : `$.employmentNav.results[0].jobInfoNav.results[0].departmentNav.name_localized`
   * Nouveau JSONPath : `$.employmentNav.results[-1:].jobInfoNav.results[0].departmentNav.name_localized`
1. Redémarrer l’approvisionnement. 

Cette modification de schéma prend également en charge le scénario de conversion de travail. 

### <a name="handling-global-assignment-scenario"></a>Gestion du scénario d’attribution globale

Lorsqu’un utilisateur dans le Centre des employés est traité pour une attribution globale, SuccessFactors ajoute une nouvelle entité *EmpEmployment* définit *assignmentClass* sur « GA ». Il crée également une nouvelle entité *Utilisateur*. L’utilisateur dispose désormais de :
* Une entité *EmpEmployment* + *Utilisateur* qui correspond à l’affectation de domicile avec *assignmentClass* défini sur « ST » et 
* Une entité *EmpEmployment* + *Utilisateur* qui correspond à l’affectation globale avec *assignmentClass* défini sur « GA ».

Pour récupérer les attributs appartenant au profil d’affectation standard et à l’utilisateur d’attribution globale, suivez les étapes ci-dessous : 

1. Ouvrez le panneau mappage d’attributs de votre application d’approvisionnement de SuccessFactors. 
1. Faites défiler vers le bas et cliquez sur **Afficher les options avancées .** .
1. Cliquez sur le lien **Passez en revue votre schéma ici** pour ouvrir l’éditeur de schéma.   
1. Cliquez sur le lien **Télécharger** pour enregistrer une copie du schéma avant de le modifier.   
1. Dans l’éditeur de schéma, appuyez sur Ctrl-H pour ouvrir le contrôle rechercher/remplacer.
1. Dans la zone de texte Rechercher, copiez et collez la valeur `$.employmentNav.results[0]`
1. Dans la zone de texte Remplacer, copiez et collez la valeur `$.employmentNav.results[?(@.assignmentClass == 'ST')]` 
1. Cliquez sur l’option « remplacer tout » pour mettre à jour le schéma. 
1. Enregistrez le schéma. 
1. Le processus ci-dessus met à jour toutes les expressions JSONPath comme suit : 
   * Ancien JSONPath : `$.employmentNav.results[0].jobInfoNav.results[0].departmentNav.name_localized`
   * Nouveau JSONPath : `$.employmentNav.results[?(@.assignmentClass == 'ST')].jobInfoNav.results[0].departmentNav.name_localized`
1. Rechargez le panneau mappage d’attributs de l’application. 
1. Faites défiler vers le bas et cliquez sur **Afficher les options avancées .** .
1. Cliquez sur **Modifier la liste des attributs de SuccessFactors**.
1. Ajoutez de nouveaux attributs pour extraire les données d’affectation globale. Par exemple, si vous souhaitez extraire le nom de service associé à un profil d’attribution global, vous pouvez ajouter l’attribut *globalAssignmentDepartment* avec l’expression JSONPath définie sur `$.employmentNav.results[?(@.assignmentClass == 'GA')].jobInfoNav.results[0].departmentNav.name_localized`. 
1. Vous pouvez maintenant transmettre les deux valeurs de service aux attributs Active Directory ou transmettre de manière sélective une valeur à l’aide du mappage d’expression. Exemple : l’expression ci-dessous définit la valeur de l’attribut AD *Département* sur *globalAssignmentDepartment* si elle est présente, sinon elle définit la valeur sur *Département* associée à l’affectation standard. 
   * `IIF(IsPresent([globalAssignmentDepartment]),[globalAssignmentDepartment],[department])`

1. Sauvegardez le mappage. 
1. Redémarrer l’approvisionnement. 

### <a name="handling-concurrent-jobs-scenario"></a>Gestion du scénario de tâches simultanées

Lorsqu’un utilisateur dans le Centre des employés a des tâches simultanées/multiples, il existe deux entités *EmpEmployment* et *Utilisateur* avec *assignmentClass* défini sur « ST ». Pour récupérer les attributs appartenant aux deux tâches, utilisez les étapes ci-dessous : 

1. Ouvrez le panneau mappage d’attributs de votre application d’approvisionnement de SuccessFactors. 
1. Faites défiler vers le bas et cliquez sur **Afficher les options avancées .** .
1. Cliquez sur **Modifier la liste des attributs de SuccessFactors**.
1. Supposons que vous souhaitez extraire le service associé à la tâche 1 et à la tâche 2. L’attribut prédéfini *département* extrait déjà la valeur de département pour la première tâche. Vous pouvez définir un nouvel attribut appelé *secondJobDepartment* et définir l’expression JSONPath sur `$.employmentNav.results[1].jobInfoNav.results[0].departmentNav.name_localized`
1. Vous pouvez maintenant transmettre les deux valeurs de service aux attributs Active Directory ou transmettre de manière sélective une valeur à l’aide du mappage d’expression. 
1. Sauvegardez le mappage. 
1. Redémarrer l’approvisionnement. 

## <a name="writeback-scenarios"></a>Scénarios d'écriture différée

Cette section aborde différents scénarios d’écriture différée. Elle recommande des approches de configuration basées sur la façon dont la messagerie et le numéro de téléphone sont configurés dans SuccessFactors.

### <a name="supported-scenarios-for-phone-and-email-write-back"></a>Scénarios pris en charge pour l’écriture différée par téléphone et par courrier électronique 

| \# | Scénario requis | Adresse de messagerie principale <br> valeur d’indicateur | Téléphone professionnel <br> valeur de l’indicateur principal | Téléphone portable <br> valeur de l’indicateur principal | Téléphone professionnel <br> mapping | Téléphone portable <br> mapping |
|--|--|--|--|--|--|--|
| 1 | * Définissez uniquement la messagerie professionnelle comme principale. <br> * Ne définissez pas de numéro de téléphone. | true | true | false | \[Non défini\] | \[Non défini\] | 
| 2 | * Dans SuccessFactors, la messagerie professionnelle et le téléphone professionnel sont principaux <br> * Faites toujours passer le numéro de téléphone d'Azure AD au téléphone professionnel et le portable au téléphone cellulaire. | true | true | false | telephoneNumber | mobile | 
| 3 | * Dans SuccessFactors, la messagerie professionnelle et le téléphone cellulaire sont principaux <br> * Faites toujours passer le numéro de téléphone d'Azure AD au téléphone professionnel et le portable au téléphone cellulaire. | true | false | true |  telephoneNumber | mobile | 
| 4 | * Dans SuccessFactors, la messagerie professionnelle est principale <br> * Dans Azure AD, vérifiez si le numéro de téléphone professionnel est présent, le cas échéant, vérifiez si le numéro de téléphone mobile est également présent, marquez le numéro de téléphone professionnel comme principal uniquement si le numéro de téléphone mobile n’est pas présent. | true | Utiliser le mappage d’expression :`IIF(IsPresent([telephoneNumber]), IIF(IsPresent([mobile]),"false", "true"), "false")` | Utiliser le mappage d’expression :`IIF(IsPresent([mobile]),"false", "true")` | telephoneNumber | mobile | 
| 5 | * Dans SuccessFactors, la messagerie professionnelle et le téléphone professionnel sont principaux <br> * Dans Azure AD, si mobile est disponible, définissez-le en tant que téléphone professionnel, sinon utilisez telephoneNumber. | true | true | false | `IIF(IsPresent([mobile]), [mobile], [telephoneNumber])` | \[Non défini\] | 

* S’il n’existe aucun mappage pour le numéro de téléphone dans le mappage d’attribut d’écriture différée, seul le courrier électronique est inclus dans l’écriture différée.
* Lors de l’intégration d’un nouvel employé dans le Centre des employés, la messagerie professionnelle et le numéro de téléphone peuvent ne pas être disponibles. Si la définition de la messagerie d’entreprise et du téléphone professionnel comme principale est obligatoire lors de l’intégration, vous pouvez définir une valeur factice pour le téléphone professionnel et la messagerie pendant la création du nouvel employé, qui sera ensuite mise à jour par l’application en écriture différée.
 
### <a name="unsupported-scenarios-for-phone-and-email-write-back"></a>Scénarios non-pris en charge pour l’écriture différée par téléphone et par courrier électronique

* Dans le Centre des employés, pendant l’intégration, la messagerie et le téléphone personnel sont définis comme principaux. L’application en écriture différée ne peut pas changer ce paramètre et définir la messagerie professionnelle et le téléphone professionnel comme principal.
* Dans le Centre des employés, le téléphone professionnel est défini comme principal. L’application en écriture différée ne peut pas changer et définir le téléphone portable comme principal.
* L’application en écriture différée ne peut pas lire les paramètres actuels de l’indicateur principal et utiliser les mêmes valeurs pour l’opération d’écriture. Les valeurs d’indicateur configurées dans le mappage d’attributs sont toujours utilisées. 

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment configurer l’approvisionnement SuccessFactors vers Active Directory](../saas-apps/sap-successfactors-inbound-provisioning-tutorial.md)
* [Découvrez comment configurer l’écriture différée vers SuccessFactors](../saas-apps/sap-successfactors-writeback-tutorial.md)
* [En savoir plus sur les attributs SuccessFactors pris en charge pour le provisionnement entrant](sap-successfactors-attribute-reference.md)










