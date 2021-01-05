---
title: Informations de référence sur les attributs SAP SuccessFactors
description: Découvrez quels attributs de SuccessFactors sont pris en charge par le provisionnement piloté par SuccessFactors-HR
services: active-directory
author: cmmdesai
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: reference
ms.workload: identity
ms.date: 08/05/2020
ms.author: chmutali
ms.reviewer: celested
ms.openlocfilehash: ef2da377c7720cfb7b431d1ce0fed56656a2b8c7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87808505"
---
# <a name="sap-successfactors-attribute-reference"></a>Informations de référence sur les attributs SAP SuccessFactors

Dans cet article, vous trouverez des informations sur les éléments suivants :

- [Entités et attributs SuccessFactors](#supported-successfactors-entities-and-attributes)
- [Mappage d’attributs par défaut](#default-attribute-mapping)

## <a name="supported-successfactors-entities-and-attributes"></a>Entités et attributs SuccessFactors pris en charge

Le tableau ci-dessous capture la liste des attributs SuccessFactors inclus par défaut dans les deux applications de provisionnement suivantes :

- [Provisionnement d’utilisateurs SuccessFactors vers Active Directory](../saas-apps/sap-successfactors-inbound-provisioning-tutorial.md)
- [Provisionnement d’utilisateurs SuccessFactors vers Azure AD](../saas-apps/sap-successfactors-inbound-provisioning-cloud-only-tutorial.md)

Reportez-vous à la [Référence sur l’intégration de SAP SuccessFactors](./sap-successfactors-integration-reference.md#retrieving-additional-attributes) pour étendre le schéma des attributs supplémentaires. 

| \# | Entité SuccessFactors                  | Attribut SuccessFactors     | Type d’opération |
|----|----------------------------------------|------------------------------|----------------|
| 1  | PerPerson                              | personIdExternal             | Lire           |
| 2  | PerPerson                              | personId                     | Lire           |
| 3  | PerPerson                              | perPersonUuid                | Lire           |
| 4  | PerPersonal                            | displayName                  | Lire           |
| 5  | PerPersonal                            | firstName                    | Lire           |
| 6  | PerPersonal                            | gender                       | Lire           |
| 7  | PerPersonal                            | lastName                     | Lire           |
| 8  | PerPersonal                            | middleName                   | Lire           |
| 9  | PerPersonal                            | preferredName                | Lire           |
| 10 | Utilisateur                                   | addressLine1                 | Lire           |
| 11 | Utilisateur                                   | addressLine2                 | Lire           |
| 12 | Utilisateur                                   | addressLIne3                 | Lire           |
| 13 | Utilisateur                                   | businessPhone                | Lire           |
| 14 | Utilisateur                                   | cellPhone                    | Lire           |
| 15 | Utilisateur                                   | city                         | Lire           |
| 16 | Utilisateur                                   | country                      | Lire           |
| 17 | Utilisateur                                   | custom01                     | Lire           |
| 18 | Utilisateur                                   | custom02                     | Lire           |
| 19 | Utilisateur                                   | custom03                     | Lire           |
| 20 | Utilisateur                                   | custom04                     | Lire           |
| 21 | Utilisateur                                   | custom05                     | Lire           |
| 22 | Utilisateur                                   | custom06                     | Lire           |
| 23 | Utilisateur                                   | custom07                     | Lire           |
| 24 | Utilisateur                                   | custom08                     | Lire           |
| 25 | Utilisateur                                   | custom09                     | Lire           |
| 26 | Utilisateur                                   | custom10                     | Lire           |
| 27 | Utilisateur                                   | custom11                     | Lire           |
| 28 | Utilisateur                                   | custom12                     | Lire           |
| 29 | Utilisateur                                   | custom13                     | Lire           |
| 30 | Utilisateur                                   | custom14                     | Lire           |
| 31 | Utilisateur                                   | empId                        | Lire           |
| 32 | Utilisateur                                   | homePhone                    | Lire           |
| 33 | Utilisateur                                   | jobFamily                    | Lire           |
| 34 | Utilisateur                                   | nickname                     | Lire           |
| 35 | Utilisateur                                   | state                        | Lire           |
| 36 | Utilisateur                                   | timeZone                     | Lire           |
| 37 | Utilisateur                                   | username                     | Lire           |
| 38 | Utilisateur                                   | zipCode                      | Lire           |
| 39 | PerPhone                               | areaCode                     | Lire           |
| 40 | PerPhone                               | countryCode                  | Lire           |
| 41 | PerPhone                               | extension                    | Lire           |
| 42 | PerPhone                               | phoneNumber                  | Lire           |
| 43 | PerPhone                               | phoneType                    | Lire           |
| 44 | PerEmail                               | emailAddress                 | Lire, Écrire    |
| 45 | PerEmail                               | emailType                    | Lire           |
| 46 | EmpEmployment                          | firstDateWorked              | Lire           |
| 47 | EmpEmployment                          | lastDateWorked               | Lire           |
| 48 | EmpEmployment                          | userId                       | Lire           |
| 49 | EmpEmployment                          | isContingentWorker           | Lire           |
| 50 | EmpJob                                 | countryOfCompany             | Lire           |
| 51 | EmpJob                                 | emplStatus                   | Lire           |
| 52 | EmpJob                                 | endDate                      | Lire           |
| 53 | EmpJob                                 | startDate                    | Lire           |
| 54 | EmpJob                                 | jobTitle                     | Lire           |
| 55 | EmpJob                                 | position                     | Lire           |
| 65 | EmpJob                                 | customString13               | Lire           |
| 56 | EmpJob                                 | managerId                    | Lire           |
| 57 | EmpJob\.BusinessUnit                   | businessUnit                 | Lire           |
| 58 | EmpJob\.BusinessUnit                   | businessUnitId               | Lire           |
| 59 | EmpJob\.Company                        | société                      | Lire           |
| 60 | EmpJob\.Company                        | companyId                    | Lire           |
| 61 | EmpJob\.Company\.CountryOfRegistration | twoCharCountryCode           | Lire           |
| 62 | EmpJob\.CostCenter                     | costCenter                   | Lire           |
| 63 | EmpJob\.CostCenter                     | costCenterId                 | Lire           |
| 64 | EmpJob\.CostCenter                     | costCenterDescription        | Lire           |
| 65 | Service\.EmpJob                     | department                   | Lire           |
| 66 | Service\.EmpJob                     | departmentId                 | Lire           |
| 67 | Division\.EmpJob                       | division                     | Lire           |
| 68 | Division\.EmpJob                       | divisionId                   | Lire           |
| 69 | EmpJob\.JobCode                        | jobCode                      | Lire           |
| 70 | EmpJob\.JobCode                        | jobCodeId                    | Lire           |
| 71 | EmpJob\.Location                       | LocationName                 | Lire           |
| 72 | EmpJob\.Location                       | officeLocationAddress        | Lire           |
| 73 | EmpJob\.Location                       | officeLocationCity           | Lire           |
| 74 | EmpJob\.Location                       | officeLocationCustomString4  | Lire           |
| 75 | EmpJob\.Location                       | officeLocationZipCode        | Lire           |
| 76 | EmpJob\.PayGrade                       | payGrade                     | Lire           |
| 77 | EmpEmploymentTermination               | activeEmploymentsCount       | Lire           |
| 78 | EmpEmploymentTermination               | latestTerminationDate        | Lire           |

## <a name="default-attribute-mapping"></a>Mappage d’attributs par défaut

Le tableau ci-dessous fournit le mappage des attributs par défaut entre les attributs SuccessFactors listés ci-dessus et les attributs AD/Azure AD. Dans le panneau « Mappage » de l’application de provisionnement Azure AD, vous pouvez modifier ce mappage par défaut pour inclure les attributs de la liste ci-dessus. 

| \# | Entité SuccessFactors                  | Attribut SuccessFactors | Mappage des attributs AD/Azure AD par défaut   | Remarque sur le traitement                                                                            |
|----|----------------------------------------|--------------------------|-----------------------------------------|----------------------------------------------------------------------------------------------|
| 1  | PerPerson                              | personIdExternal         | employeeId                              | Utilisé comme attribut correspondant                                                                   |
| 2  | PerPerson                              | perPersonUuid            | \[Non mappé \- utilisé comme ancre source\] | Pendant la synchronisation initiale, le service de provisionnement lie l’élément personUuid à un élément objectGuid\ existant.  |
| 3  | PerPersonal                            | displayName              | displayName                             | N/D                                                                                           |
| 4  | PerPersonal                            | firstName                | givenName                               | N/D                                                                                           |
| 5  | PerPersonal                            | lastName                 | sn                                      | N/D                                                                                           |
| 6  | Utilisateur                                   | addressLine1             | streetAddress                           | N/D                                                                                           |
| 7  | Utilisateur                                   | city                     | l                                       | N/D                                                                                           |
| 8  | Utilisateur                                   | country                  | co                                      | N/D                                                                                           |
| 9  | Utilisateur                                   | state                    | st                                      | N/D                                                                                           |
| 10 | Utilisateur                                   | username                 | samAccountName                          | N/D                                                                                           |
| 11 | Utilisateur                                   | zipCode                  | postalCode                              | N/D                                                                                           |
| 12 | PerEmail                               | emailAddress             | mail                                    | N/D                                                                                           |
| 13 | EmpJob                                 | jobTitle                 | title                                   | N/D                                                                                           |
| 14 | EmpJob                                 | managerId                | manager                                 | N/D                                                                                           |
| 15 | EmpJob\.Company\.CountryOfRegistration | twoCharCountryCode       | c                                       | N/D                                                                                           |
| 16 | Service\.EmpJob                     | department               | department                              | N/D                                                                                           |
| 17 | Division\.EmpJob                       | division                 | société                                 | N/D                                                                                           |
| 18 | EmpJob\.Location                       | officeLocationAddress    | streetAddress                           | N/D                                                                                           |
| 19 | EmpJob\.Location                       | officeLocationZipCode    | postalCode                              | N/D                                                                                           |
| 20 | EmpEmploymentTermination               | activeEmploymentsCount   | accountEnabled                          | Si activeEmploymentsCount=0, désactivez account\.                                           |
