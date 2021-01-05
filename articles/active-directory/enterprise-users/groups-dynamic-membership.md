---
title: Règles d’appartenance aux groupes remplis dynamiquement - Azure AD | Microsoft Docs
description: Comment créer des règles d’appartenance pour remplir automatiquement des groupes, et documentation de référence sur les règles.
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.subservice: enterprise-users
ms.workload: identity
ms.topic: overview
ms.date: 12/02/2020
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: c60d54a905f460eb5c26c2f183cd22b175a5b3c4
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96860811"
---
# <a name="dynamic-membership-rules-for-groups-in-azure-active-directory"></a>Règles d’appartenance de groupe dynamique dans Azure Active Directory

Dans Azure Active Directory (Azure AD), vous pouvez créer des règles complexes basées sur des attributs pour activer des appartenances dynamiques pour des groupes. L’appartenance de groupe dynamique réduit la charge administrative d’ajout et de suppression d’utilisateurs. Cet article détaille les propriétés et la syntaxe à utiliser pour créer des règles d’appartenance dynamique pour des utilisateurs ou des appareils. Vous pouvez définir une règle d’appartenance dynamique sur les groupes de sécurité ou Microsoft 365.

Lorsqu’un attribut d’un utilisateur ou d’un appareil change, le système évalue toutes les règles de groupe dynamique d’un annuaire pour voir si la modification déclenche des ajouts ou suppressions de groupe. Si un utilisateur ou un appareil respecte une règle d’un groupe, il est ajouté en tant que membre de ce groupe. S’il ne respecte plus la règle, il est supprimé. Vous ne pouvez pas ajouter ni supprimer manuellement un membre dans un groupe dynamique.

- Vous pouvez créer un groupe dynamique pour les appareils ou utilisateurs, mais vous ne pouvez pas créer une règle qui contient à la fois des utilisateurs et des appareils.
- Vous ne pouvez pas créer un groupe d’appareils basé sur des attributs des propriétaires d’appareils. Des règles d’appartenance d’appareil ne peuvent référencer que des attributs d’appareils.

> [!NOTE]
> Cette fonctionnalité nécessite une licence Azure AD Premium P1 pour chaque utilisateur unique membre d’un ou de plusieurs groupes dynamiques. Vous n’avez pas à attribuer des licences aux utilisateurs pour qu’ils soient membres de groupes dynamiques. Toutefois, vous devez disposer du nombre minimal de licences dans l’organisation Azure AD pour couvrir tous les utilisateurs de ce type. Par exemple, si vous avez un total de 1 000 utilisateurs uniques dans tous les groupes dynamiques de votre organisation, vous devez disposer d’au moins 1 000 licences Azure AD Premium P1 pour répondre aux exigences de licence.
> Aucune licence n’est requise pour les appareils qui sont membres d’un groupe d’appareils dynamique.

## <a name="rule-builder-in-the-azure-portal"></a>Générateur de règles dans le portail Azure

Azure AD fournit un générateur de règles pour créer et mettre à jour plus rapidement vos règles importantes. Le générateur de règles prend en charge la construction de jusqu’à cinq expressions. Le générateur de règles facilite la création d’une règle avec quelques expressions simples. Toutefois, il ne peut pas être utilisé pour reproduire chaque règle. Si le générateur de règles ne prend pas en charge la règle que vous souhaitez créer, vous pouvez utiliser la zone de texte.

Voici quelques exemples de règles avancées ou de syntaxe pour lesquelles nous vous recommandons de construire à l’aide de la zone de texte :

- Règle avec plus de cinq expressions
- Règle de collaborateurs directs
- Définition de la [priorité des opérateurs](#operator-precedence)
- [Règles avec des expressions complexes](#rules-with-complex-expressions), par exemple `(user.proxyAddresses -any (_ -contains "contoso"))`

> [!NOTE]
> Le générateur de règles peut ne pas être en mesure d’afficher certaines règles construites dans la zone de texte. Un message peut s’afficher lorsque le générateur de règles n’est pas en mesure d’afficher la règle. Le générateur de règles ne modifie en aucune façon la syntaxe, la validation ou le traitement des règles de groupe dynamique pris en charge.

Pour obtenir des instructions pas à pas, consultez [Créer ou mettre à jour un groupe dynamique](groups-create-rule.md).

![Ajouter une règle d’appartenance au groupe dynamique](./media/groups-dynamic-membership/update-dynamic-group-rule.png)

### <a name="rule-syntax-for-a-single-expression"></a>Syntaxe de règle pour une expression unique

Une expression unique est la forme la plus simple d’une règle d’appartenance, qui ne comprend que les trois parties précitées. Une règle avec une expression unique ressemble à ceci : `Property Operator Value`, où la syntaxe de la propriété est le nom de object.property.

Voici un exemple de règle d’appartenance correctement construite avec une expression unique :

```
user.department -eq "Sales"
```

Les parenthèses sont facultatives pour une expression unique. La longueur totale du corps de votre règle d’appartenance ne peut pas dépasser 2048 caractères.

## <a name="constructing-the-body-of-a-membership-rule"></a>Construction du corps d’une règle d’appartenance

Une règle d’appartenance qui remplit automatiquement un groupe d’utilisateurs ou d’appareils est une expression binaire qui génère un résultat vrai ou faux. Les trois parties d’une règle simple sont les suivantes :

- Propriété
- Opérateur
- Valeur

L’ordre des parties au sein d’une expression est importants pour éviter les erreurs de syntaxe.

## <a name="supported-properties"></a>Propriétés prises en charge

Il existe trois types de propriétés utilisables pour construire une règle d’appartenance.

- Boolean
- String
- Collection de chaînes

Les propriétés utilisateur que vous pouvez utiliser pour créer une expression unique sont les suivantes.

### <a name="properties-of-type-boolean"></a>Propriétés de type booléen

| Propriétés | Valeurs autorisées | Usage |
| --- | --- | --- |
| accountEnabled |true false |user.accountEnabled -eq true |
| dirSyncEnabled |true false |user.dirSyncEnabled -eq true |

### <a name="properties-of-type-string"></a>Propriétés de type chaîne

| Propriétés | Valeurs autorisées | Usage |
| --- | --- | --- |
| city |Toute valeur de chaîne ou *null* |(user.city -eq "value") |
| country |Toute valeur de chaîne ou *null* |(user.country -eq "value") |
| companyName | Toute valeur de chaîne ou *null* | (user.companyName -eq "value") |
| department |Toute valeur de chaîne ou *null* |(user.department -eq "value") |
| displayName |Valeur de chaîne quelconque |(user.displayName -eq "value") |
| employeeId |Valeur de chaîne quelconque |(user.employeeId -eq "value")<br>(user.employeeId -ne *null*) |
| facsimileTelephoneNumber |Toute valeur de chaîne ou *null* |(user.facsimileTelephoneNumber -eq "value") |
| givenName |Toute valeur de chaîne ou *null* |(user.givenName -eq "value") |
| jobTitle |Toute valeur de chaîne ou *null* |(user.jobTitle -eq "value") |
| mail |Toute valeur de chaîne ou *null* (adresse SMTP de l’utilisateur) |(user.mail -eq "value") |
| mailNickName |Toute valeur de chaîne (alias de messagerie de l’utilisateur) |(user.mailNickName -eq "value") |
| mobile |Toute valeur de chaîne ou *null* |(user.mobile -eq "value") |
| objectId |GUID de l’objet utilisateur |(user.objectId -eq "11111111-1111-1111-1111-111111111111") |
| onPremisesSecurityIdentifier | Identificateur de sécurité (SID) local pour les utilisateurs synchronisés localement vers le cloud. |(user.onPremisesSecurityIdentifier -eq "S-1-1-11-1111111111-1111111111-1111111111-1111111") |
| passwordPolicies |Aucune DisableStrongPassword DisablePasswordExpiration DisablePasswordExpiration, DisableStrongPassword |(user.passwordPolicies -eq "DisableStrongPassword") |
| physicalDeliveryOfficeName |Toute valeur de chaîne ou *null* |(user.physicalDeliveryOfficeName -eq "value") |
| postalCode |Toute valeur de chaîne ou *null* |(user.postalCode -eq "value") |
| preferredLanguage |Code ISO 639-1 |(user.preferredLanguage -eq "en-US") |
| sipProxyAddress |Toute valeur de chaîne ou *null* |(user.sipProxyAddress -eq "value") |
| state |Toute valeur de chaîne ou *null* |(user.state -eq "value") |
| streetAddress |Toute valeur de chaîne ou *null* |(user.streetAddress -eq "value") |
| surname |Toute valeur de chaîne ou *null* |(user.surname -eq "value") |
| telephoneNumber |Toute valeur de chaîne ou *null* |(user.telephoneNumber -eq "value") |
| usageLocation |Code de pays/région à deux lettres |(user.usageLocation -eq "US") |
| userPrincipalName |Valeur de chaîne quelconque |(user.userPrincipalName -eq "alias@domain") |
| userType |member guest *null* |(user.userType -eq "Member") |

### <a name="properties-of-type-string-collection"></a>Propriétés de type collection de chaînes

| Propriétés | Valeurs autorisées | Usage |
| --- | --- | --- |
| otherMails |Valeur de chaîne quelconque |(user.otherMails -contains "alias@domain") |
| proxyAddresses |SMTP: alias@domain smtp: alias@domain |(user.proxyAddresses -contains "SMTP: alias@domain") |

Concernant les propriétés utilisées pour les règles d’appareils, voir [Règles pour les appareils](#rules-for-devices).

## <a name="supported-expression-operators"></a>Opérateurs d’expression pris en charge

Le tableau suivant répertorie tous les opérateurs pris en charge et leur syntaxe pour une expression unique. Les opérateurs peuvent être utilisés avec ou sans le préfixe de trait d’union (-).

| Opérateur | Syntaxe |
| --- | --- |
| Non égal à |-ne |
| Égal à |-eq |
| Ne commence pas par |-notStartsWith |
| Starts With |-startsWith |
| Ne contient pas |-notContains |
| Contient |-contains |
| Ne correspond pas |-notMatch |
| Correspond |-match |
| Dans | -in |
| Pas dans | -notIn |

### <a name="using-the--in-and--notin-operators"></a>Utilisation des opérateurs -in et -notIn

Si vous souhaitez comparer la valeur d’un attribut utilisateur par rapport à un nombre de valeurs différentes, vous pouvez utiliser les opérateurs -in ou -notIn. Insérez les symboles de crochet « [ » et «] » au début et à la fin de la liste de valeurs.

 Dans l’exemple suivant, l’expression est vraie si la valeur de user.department est égale à l’une des valeurs dans la liste :

```
   user.department -in ["50001","50002","50003","50005","50006","50007","50008","50016","50020","50024","50038","50039","51100"]
```


### <a name="using-the--match-operator"></a>Utilisation de l’opérateur -match 
L’opérateur **-match** est utilisé pour la correspondance de n’importe quelle expression régulière. Exemples :

```
user.displayName -match "Da.*"   
```
DA, Dav, David retourne true, aDa retourne false.

```
user.displayName -match ".*vid"
```
David retourne true, Da retourne false.

## <a name="supported-values"></a>Valeurs prises en charge

Les valeurs utilisées dans une expression peuvent être de plusieurs types, à savoir :

* Chaînes
* Booléen : « True », « False »
* Nombres
* Tableaux : tableau de nombres, tableau de chaînes

Lorsque vous spécifiez une valeur dans une expression, il est important d’utiliser la syntaxe correcte pour éviter les erreurs. Voici quelques conseils de syntaxe :

* Les guillemets doubles sont facultatifs, sauf si la valeur est une chaîne.
* Les opérations de chaîne et regex (expressions régulières) ne prennent pas en compte la casse.
* Quand une valeur de chaîne contient des guillemets doubles, les deux guillemets doivent être échappés à l’aide du caractère \`. Par exemple, user.department -eq \`"Sales\`" est la syntaxe appropriée quand « Sales » est la valeur.
* Vous pouvez également effectuer des vérifications de valeur Null, en utilisant null en tant que valeur. Par exemple, `user.department -eq null`.

### <a name="use-of-null-values"></a>Utiliser des valeurs Null

Pour spécifier une valeur null dans une règle, vous pouvez utiliser la valeur *null*. 

* Utilisez -eq ou -ne lors de la comparaison de la valeur *null* dans une expression.
* N’insérez des guillemets autour du mot *null* que si vous voulez qu’il soit interprété comme une valeur de chaîne littérale.
* L’opérateur -not ne peut pas être utilisé comme un opérateur de comparaison pour la valeur null. Si vous l’utilisez, vous obtenez une erreur, que vous utilisiez une valeur null ou $null.

La manière correcte de référencer la valeur null est la suivante :

```
   user.mail –ne null
```

## <a name="rules-with-multiple-expressions"></a>Règles comportant plusieurs expressions

Une règle d’appartenance de groupe peut se composer de plusieurs expressions uniques reliées par les opérateurs logiques -and, -or et -not. Des opérateurs logiques peuvent également être utilisés en combinaison. 

Voici des exemples de règles d’appartenance correctement construites avec plusieurs expressions :

```
(user.department -eq "Sales") -or (user.department -eq "Marketing")
(user.department -eq "Sales") -and -not (user.jobTitle -contains "SDE")
```

### <a name="operator-precedence"></a>Précédence des opérateurs

Tous les opérateurs sont répertoriés ci-dessous par ordre de priorité, du plus élevé au plus bas. Les opérateurs figurant sur une même ligne ont une priorité identique :

```
-eq -ne -startsWith -notStartsWith -contains -notContains -match –notMatch -in -notIn
-not
-and
-or
-any -all
```

Voici un exemple de priorité d’opérateur où deux expressions sont évaluées pour l’utilisateur :

```
   user.department –eq "Marketing" –and user.country –eq "US"
```

Des parenthèses ne sont nécessaires que lorsque la priorité ne répond pas à vos besoins. Par exemple, si vous souhaitez que department soit évalué en premier, ce qui suit montre comment utiliser des parenthèses pour déterminer l’ordre :

```
   user.country –eq "US" –and (user.department –eq "Marketing" –or user.department –eq "Sales")
```

## <a name="rules-with-complex-expressions"></a>Règles avec des expressions complexes

Une règle d’appartenance peut être constituée d’expressions complexes où les propriétés, les opérateurs et les valeurs prennent des formes plus complexes. Des expressions sont considérées comme complexes quand l’une des conditions suivantes est vraie :

* La propriété consiste en une collection de valeurs, plus précisément, des propriétés à valeurs multiples
* Les expressions utilisent les opérateurs -any et -all
* La valeur de l’expression peut être elle-même une ou plusieurs expressions

## <a name="multi-value-properties"></a>Propriétés à valeurs multiples

Les propriétés à valeurs multiples sont des collections d’objets du même type. Vous pouvez vous en servir pour créer des règles d’appartenance utilisant les opérateurs -any et -all.

| Propriétés | Valeurs | Usage |
| --- | --- | --- |
| assignedPlans | Chaque objet de la collection affiche les propriétés de chaînes suivantes : capabilityStatus, service, servicePlanId |user.assignedPlans -any (assignedPlan.servicePlanId -eq "efb87545-963c-4e0d-99df-69c6916d9eb0" -et assignedPlan.capabilityStatus -eq "Enabled") |
| proxyAddresses| SMTP: alias@domain smtp: alias@domain | (user.proxyAddresses -any (\_ -contains "contoso")) |

### <a name="using-the--any-and--all-operators"></a>Utilisation des opérateurs -any et -all

Vous pouvez utiliser les opérateurs -any et -all pour appliquer respectivement une condition à un ou tous les objets de la collection.

* -any (respectée lorsqu’au moins un élément de la collection correspond à la condition)
* -all (respectée lorsque tous les éléments de la collection correspondent à la condition)

#### <a name="example-1"></a>Exemple 1

assignedPlans est une propriété à valeurs multiples qui répertorie tous les plans de service assignés à l’utilisateur. L’expression ci-dessous sélectionne les utilisateurs qui ont le plan de service (en tant que valeur GUID) Exchange Online (Plan 2), également en état Activé :

```
user.assignedPlans -any (assignedPlan.servicePlanId -eq "efb87545-963c-4e0d-99df-69c6916d9eb0" -and assignedPlan.capabilityStatus -eq "Enabled")
```

Vous pouvez utiliser une règle telle que celle-ci afin de regrouper tous les utilisateurs pour lesquels une fonctionnalité Microsoft 365 (ou un autre service en ligne Microsoft) est activée. Vous pourriez ensuite l’appliquer au groupe avec un ensemble de stratégies.

#### <a name="example-2"></a>Exemple 2

L’expression suivante sélectionne tous les utilisateurs qui disposent d’un plan de service associé au service Intune (identifié par le nom de service « SCO ») :

```
user.assignedPlans -any (assignedPlan.service -eq "SCO" -and assignedPlan.capabilityStatus -eq "Enabled")
```

### <a name="using-the-underscore-_-syntax"></a>Utilisation de la syntaxe de trait de soulignement (\_)

La syntaxe de trait de soulignement (\_) correspond aux occurrences d’une valeur spécifique dans une des propriétés de collection de chaîne à valeurs multiples pour ajouter des utilisateurs ou des appareils à un groupe dynamique. Elle est utilisée avec les opérateurs -any ou -all.

Voici un exemple d’utilisation du trait de soulignement (\_) dans une règle pour ajouter des membres à partir de user.proxyAddress (le même principe s’applique pour user.otherMails). Cette règle ajoute au groupe n’importe quel utilisateur avec l’adresse de proxy contenant « contoso ».

```
(user.proxyAddresses -any (_ -contains "contoso"))
```

## <a name="other-properties-and-common-rules"></a>Autres propriétés et les règles communes

### <a name="create-a-direct-reports-rule"></a>Créer une règle « Collaborateurs directs »

Vous pouvez créer un groupe contenant tous les collaborateurs directs d’un responsable. À l’avenir, lorsque les collaborateurs directs du responsable changeront, l’appartenance du groupe sera ajustée automatiquement.

La règle de collaborateurs directs est construite à l’aide de la syntaxe suivante :

```
Direct Reports for "{objectID_of_manager}"
```

Voici un exemple de règle valide où « 62e19b97-8b3d-4d4a-a106-4ce66896a863 » est l’ID d’objet du responsable :

```
Direct Reports for "62e19b97-8b3d-4d4a-a106-4ce66896a863"
```

Les conseils suivants peuvent vous aider à utiliser la règle correctement.

- **Manager ID** est l’ID d’objet du responsable. Il figure dans le **Profil** du responsable.
- Pour que la règle fonctionne, vérifiez que la propriété **Manager** est correctement définie pour les utilisateurs de votre organisation. Vous pouvez vérifier la valeur actuelle dans le **Profil** de l’utilisateur.
- Cette règle prend en charge uniquement les collaborateurs directs du responsable. En d’autres termes, vous ne peut pas créer de groupe avec les collaborateurs directs du responsable *et* leurs collaborateurs.
- Cette règle ne peut pas être combinée avec d’autres règles d’appartenance.

### <a name="create-an-all-users-rule"></a>Créer une règle « Tous les utilisateurs »

Vous pouvez créer un groupe contenant tous les utilisateurs d’une organisation à l’aide d’une règle d’appartenance. Lors de l’ajout ou de la suppression d’utilisateurs de l’organisation, l’appartenance du groupe est ajustée automatiquement.

La règle « Tous les utilisateurs » est construite à l’aide d’une expression unique utilisant l’opérateur -ne et la valeur null. Cette règle ajoute au groupe les utilisateurs invités B2B, ainsi que les utilisateurs membres.

```
user.objectId -ne null
```
Si vous souhaitez que votre groupe exclue les utilisateurs invités et inclue uniquement les membres de votre organisation, vous pouvez utiliser la syntaxe suivante :

```
(user.objectId -ne null) -and (user.userType -eq "Member")
```

### <a name="create-an-all-devices-rule"></a>Créer une règle « Tous les appareils »

Vous pouvez créer un groupe contenant tous les appareils d’une organisation à l’aide d’une règle d’appartenance. Lors de l’ajout ou de la suppression d’appareils de l’organisation, l’appartenance du groupe est ajustée automatiquement.

La règle « Tous les appareils » est construite à l’aide d’une expression unique utilisant l’opérateur -ne et la valeur null :

```
device.objectId -ne null
```

## <a name="extension-properties-and-custom-extension-properties"></a>Attributs d’extension et propriétés d’extension personnalisée

Les attributs d’extension et les propriétés d’extension personnalisées sont pris en charge en tant que propriétés de chaîne dans les règles d’appartenance dynamique. Les [attributs d’extension](/graph/api/resources/onpremisesextensionattributes) sont synchronisés à partir de Windows Server AD local et prennent le format « ExtensionAttributeX », où X a une valeur de 1 à 15. Voici en exemple de règle utilisant un attribut d’extension en tant que propriété :

```
(user.extensionAttribute15 -eq "Marketing")
```

Les [propriétés d’extension personnalisées](../hybrid/how-to-connect-sync-feature-directory-extensions.md) sont synchronisées à partir de Windows Server AD local ou à partir d’une application SaaS connectée, et leur format est `user.extension_[GUID]_[Attribute]`, où :

* [GUID] est l’identificateur unique dans Azure AD pour l’application qui a créé la propriété dans Azure AD
* [Attribute] est le nom attribué à la propriété lors de sa création

Voici un exemple de règle utilisant une propriété d’extension personnalisée :

```
user.extension_c272a57b722d4eb29bfe327874ae79cb_OfficeNumber -eq "123"
```

Vous pouvez accéder au nom de la propriété personnalisée dans le répertoire en lançant une requête sur une propriété d’utilisateur à l’aide d’Afficheur Graph, et en recherchant le nom de propriété. De plus, vous pouvez désormais sélectionner le lien **Obtenir des propriétés d’extension personnalisées** dans le générateur de règles de groupe d’utilisateurs dynamiques pour entrer un ID d’application unique et recevoir la liste complète des propriétés d’extension personnalisées à utiliser lors de la création d’une règle d’appartenance dynamique. Vous pouvez également actualiser cette liste afin d’obtenir les nouvelles propriétés d’extension personnalisées pour cette application.

## <a name="rules-for-devices"></a>Règles pour les appareils

Vous pouvez également créer une règle qui sélectionne des objets d’appareil pour l’appartenance à un groupe. Vous ne pouvez pas avoir à la fois des utilisateurs et des appareils en tant que membres du groupe. 

> [!NOTE]
> L’attribut **organizationalUnit** n’est plus répertorié et ne doit pas être utilisé. Cette chaîne est définie par Intune dans des cas spécifiques mais n’est pas reconnue par Azure AD, donc aucun périphérique n’est ajouté aux groupes en fonction de cet attribut.

> [!NOTE]
> systemlabels est un attribut en lecture seule qui ne peut pas être défini avec Intune.
>
> Pour Windows 10, le format correct de l’attribut deviceOSVersion est le suivant : (device.deviceOSVersion -eq "10.0.17763"). La mise en forme peut être validée avec la cmdlet PowerShell Get-MsolDevice.

Les attributs d’appareil suivants peuvent être utilisés.

 Attribut d’appareil  | Valeurs | Exemple
 ----- | ----- | ----------------
 accountEnabled | true false | (device.accountEnabled -eq true)
 displayName | Toute valeur de chaîne. |(device.displayName -eq "Rob iPhone")
 deviceOSType | Toute valeur de chaîne. | (device.deviceOSType -eq "iPad") ou (device.deviceOSType -eq "iPhone")<br>(device.deviceOSType -contains "AndroidEnterprise")<br>(device.deviceOSType -eq "AndroidForWork")
 deviceOSVersion | Toute valeur de chaîne. | (device.deviceOSVersion -eq "9.1")
 deviceCategory | Un nom de catégorie d’appareil valide. | (device.deviceCategory -eq "BYOD")
 deviceManufacturer | Toute valeur de chaîne. | (device.deviceManufacturer -eq "Samsung")
 deviceModel | Toute valeur de chaîne. | (device.deviceModel -eq "iPad Air")
 deviceOwnership | Personnel, Entreprise, Inconnu | (device.deviceOwnership -eq "Company")
 enrollmentProfileName | Nom du profil d’inscription d’appareil Apple, nom du profil d’inscription d’appareil dédié appartenant à l’entreprise Android Enterprise ou nom du profil Windows Autopilot | (device.enrollmentProfileName -eq "DEP iPhones")
 isRooted | true false | (device.isRooted -eq true)
 managementType | Gestion des périphériques mobiles (pour les appareils mobiles).<br>PC (pour les ordinateurs gérés par l’agent PC Intune) | (device.managementType -eq "MDM")
 deviceId | Un ID d’appareil Azure AD valide. | (device.deviceId -eq "d4fe7726-5966-431c-b3b8-cddc8fdb717d")
 objectId | Un ID d’objet Azure AD valide. |  (device.objectId -eq "76ad43c9-32c5-45e8-a272-7b58b58f596d")
 devicePhysicalIds | toute valeur de chaîne utilisée par AutoPilot, par exemple tous les appareils AutoPilot, OrderID ou PurchaseOrderID  | (device.devicePhysicalIDs -any _ -contains "[ZTDId]") (device.devicePhysicalIds -any _ -eq "[OrderID]:179887111881") (device.devicePhysicalIds -any _ -eq "[PurchaseOrderId]:76222342342")
 systemLabels | n’importe quelle chaîne correspondant à la propriété d’appareil Intune pour baliser les appareils Modern Workplace | (device.systemLabels -contains "M365Managed")

> [!Note]  
> Quand vous créez des groupes dynamiques pour des appareils, vous devez définir deviceOwnership avec la valeur « Company ». Sur Intune, la propriété de l’appareil est représentée comme Corporate. Pour plus d’informations, consultez [OwnerTypes](/intune/reports-ref-devices#ownertypes). 

## <a name="next-steps"></a>Étapes suivantes

Ces articles fournissent des informations supplémentaires sur les groupes dans Azure Active Directory.

- [Consulter les groupes existants](../fundamentals/active-directory-groups-view-azure-portal.md)
- [Création d’un nouveau groupe et ajout de membres](../fundamentals/active-directory-groups-create-azure-portal.md)
- [Gérer les paramètres d’un groupe](../fundamentals/active-directory-groups-settings-azure-portal.md)
- [Gérer l’appartenance à un groupe](../fundamentals/active-directory-groups-membership-azure-portal.md)
- [Gérer les règles dynamiques pour les utilisateurs dans un groupe](groups-create-rule.md)
