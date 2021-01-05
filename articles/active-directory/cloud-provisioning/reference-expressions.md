---
title: Référence des fonctions et expressions de provisionnement cloud Azure AD Connect
description: reference
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 12/02/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1a775ce6b7c560783a22697c5dd92288c5d5b7d4
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96343703"
---
# <a name="writing-expressions-for-attribute-mappings-in-azure-active-directory"></a>Écriture d’expressions pour les mappages d’attributs dans Azure Active Directory
Quand vous configurez le provisionnement cloud, l’un des types de mappages d’attributs que vous pouvez spécifier est un mappage d’expression. 

Le mappage d’expression vous permet de personnaliser des attributs à l’aide d’une expression de type script.  Cela vous permet de transformer les données locales en une valeur nouvelle ou différente.  Par exemple, vous pouvez combiner deux attributs en un seul parce que cet attribut unique est utilisé par l’une de vos applications cloud.

Le document suivant couvre les expressions de type script utilisées pour transformer les données.  Cela ne constitue qu’une partie du processus.  Ensuite, vous devez utiliser cette expression et la placer dans une requête web adressée à votre locataire.  Pour plus d’informations, consultez [Transformations](how-to-transformation.md).

## <a name="syntax-overview"></a>Vue d’ensemble de la syntaxe
La syntaxe des expressions pour les mappages d’attributs rappelle celle des fonctions Visual Basic pour Applications (VBA).

* L’expression entière doit être définie en termes de fonctions, qui sont constituées d’un nom suivi d’arguments entre parenthèses : <br>
  *FunctionName(`<<argument 1>>`,`<<argument N>>`)*
* Vous pouvez imbriquer des fonctions dans d’autres. Par exemple : <br> *FunctionOne(FunctionTwo(`<<argument1>>`))*
* Vous pouvez passer trois différents types d’arguments dans des fonctions :
  
  1. Des attributs, qui doivent être placés entre crochets. Par exemple : [nom_attribut]
  2. Des constantes de chaîne, qui doivent être placées entre des guillemets doubles. Par exemple : « États-Unis »
  3. D’autres fonctions. Par exemple : FunctionOne(`<<argument1>>`, FunctionTwo(`<<argument2>>`))
* Pour les constantes de chaîne, si vous avez besoin d’une barre oblique inverse (\) ou d’un guillemet (") dans la chaîne, vous devez le faire précéder du symbole de barre oblique inverse (\). Par exemple : « Nom de l’entreprise : \\"Contoso"\\ »

## <a name="list-of-functions"></a>Liste des fonctions
| Liste des fonctions | Description |
|-----|----|
|[Append](#append)|prend une valeur de chaîne source et ajoute le suffixe à la fin de celle-ci.|
|[BitAnd](#bitand)|la fonction BitAnd définit des bits spécifiés sur une valeur.|
|[CBool](#cbool)|La fonction CBool renvoie une valeur booléenne basée sur l’expression évaluée.|
|[ConvertFromBase64](#convertfrombase64)|La fonction ConvertFromBase64 convertit la valeur encodée en base64 en chaîne régulière.|
|[ConvertToBase64](#converttobase64)|La fonction ConvertToBase64 convertit une chaîne en chaîne Unicode base64. |
|[ConvertToUTF8Hex](#converttoutf8hex)|La fonction ConvertToUTF8Hex convertit une chaîne en valeur hexadécimale encodée UTF8.|
|[Count](#count)|La fonction Count renvoie le nombre d’éléments dans un attribut à valeurs multiples.|
|[Cstr](#cstr)|La fonction CStr convertit en un type de données de chaîne.|
|[DateFromNum](#datefromnum)|La fonction DateFromNum convertit une valeur au format de date AD en un type DateTime.|
|[DNComponent](#dncomponent)|La fonction DNComponent renvoie la valeur d’un composant de nom de domaine spécifié en partant de la gauche.|
|[Error](#error)|La fonction Error sert à renvoyer une erreur personnalisée.|
|[FormatDateTime](#formatdatetime) |prend une chaîne de date dans un format et la convertit dans un autre format.| 
|[GUID](#guid)|La fonction Guid génère un nouveau GUID aléatoire.|           
|[IIF](#iif)|La fonction IIF renvoie une valeur parmi un ensemble de valeurs possibles en fonction d’une condition spécifiée.|
|[InStr](#instr)|La fonction InStr recherche la première occurrence d’une sous-chaîne dans une chaîne.|
|[IsNull](#isnull)|La fonction IsNull renvoie true si l’expression correspond à la valeur Null.|
|[IsNullOrEmpty](#isnullorempty)|La fonction IsNullOrEmpty renvoie la valeur true si l’expression a pour valeur Null ou s’il s’agit d’une chaîne vide.|         
|[IsPresent](#ispresent)|La fonction IsPresent renvoie true si l’expression correspond à une chaîne qui n’a pas la valeur Null et n’est pas vide.|    
|[IsString](#isstring)|La fonction IsString prend la valeur True si l’expression peut être évaluée en tant que type de chaîne.|
|[Item](#item)|La fonction Item renvoie un élément à partir d’une chaîne/d’un attribut à valeurs multiples.|
|[Join](#join) |Join() est similaire à Append(), mais elle peut combiner plusieurs valeurs de chaîne **sources** dans une même chaîne et chaque valeur sera séparée par une chaîne de **séparation**.| 
|[Left](#left)|La fonction Left renvoie un nombre spécifié de caractères en partant de la gauche d’une chaîne.|
|[Mid](#mid) |retourne une sous-chaîne de la valeur source. Une sous-chaîne est une chaîne qui ne contient que certains des caractères de la chaîne source.|
|[NormalizeDiacritics](#normalizediacritics)|Nécessite un argument de chaîne. Retourne la chaîne, où les caractères diacritiques ont été remplacés par leurs équivalents non diacritiques.|
|[Not](#not) |inverse la valeur booléenne de la **source**. Si la valeur **source** est « *True* », cette fonction retourne «*False* ». Sinon, elle retourne «*True*».| 
|[RemoveDuplicates](#removeduplicates)|La fonction RemoveDuplicates prend une chaîne à valeurs multiples et vérifie que chaque valeur est unique.| 
|[Replace](#replace) |Remplace les valeurs dans une chaîne. | 
|[SelectUniqueValue](#selectuniquevalue)|Nécessite un minimum de deux arguments, qui sont définis à l’aide d’expressions de règles de génération de valeur unique. La fonction évalue chaque règle, puis vérifie la valeur générée pour l’unicité dans le répertoire/application cible.| 
|[SingleAppRoleAssignment](#singleapproleassignment)|Retourne un appRoleAssignment unique parmi la liste de tous les appRoleAssignments affectés à un utilisateur pour une application donnée.| 
|[Split](#split)|Fractionne une chaîne en un tableau à plusieurs valeurs, en utilisant le caractère délimiteur spécifié.|
|[StringFromSID](#stringfromsid)|La fonction StringFromSid convertit en chaîne un tableau d’octets contenant un identificateur de sécurité.| 
|[StripSpaces](#stripspaces) |supprime tous les caractères d’espacement (" ") de la chaîne source.| 
|[Switch](#switch)|quand la valeur **source** correspond à une **clé**, retourne la **valeur** de cette **clé**. | 
|[ToLower](#tolower)|prend une valeur de la chaîne *source* et la convertit en minuscules à l’aide des règles de culture spécifiées.| 
|[ToUpper](#toupper)|prend une valeur de la chaîne *source* et la convertit en majuscule à l’aide des règles de culture spécifiées.|
|[Trim](#trim)|La fonction Trim supprime les espaces blancs situés au début et à la fin d’une chaîne.|
|[Word](#word)|La fonction Word retourne un mot contenu dans une chaîne, en fonction des paramètres qui décrivent les délimiteurs à utiliser et le nombre de mots à retourner.|

---
### <a name="append"></a>Ajouter
**Fonction :**<br> Append(source, suffixe)

**Description :**<br> prend une valeur de chaîne source et ajoute le suffixe à la fin de celle-ci.

**Paramètres :**<br> 

   | Nom | Requis / Répétition | Type | Notes |
   | --- | --- | --- | --- |
   | **source** |Obligatoire |String |Généralement le nom de l’attribut de l’objet source. |
   | **suffix** |Obligatoire |String |Chaîne que vous souhaitez ajouter à la fin de la valeur source. |

---
### <a name="bitand"></a>BitAnd
**Description :**  
la fonction BitAnd définit des bits spécifiés sur une valeur.

**Syntaxe :**  
`num BitAnd(num value1, num value2)`

* value1, value2 : valeurs numériques qui doivent être liées par AND.

**Remarques :**  
Cette fonction convertit les deux paramètres de la représentation binaire et définit un bit sur :

* 0 - si un des bits, ou les deux bits correspondants dans *value1* et *value2* ont pour valeur 0
* 1 - si les deux bits correspondants sont définis sur 1.

En d’autres termes, elle renvoie 0 dans tous les cas, sauf si les bits correspondants de ces deux paramètres sont définis sur 1.

**Exemple :**  
 
 `BitAnd(&HF, &HF7)`</br>
 Renvoie 7, car les valeurs hexadécimales « F » ET « F7 » donnent cette valeur.

---

### <a name="cbool"></a>CBool
**Description :**  
La fonction CBool renvoie une valeur booléenne basée sur l’expression évaluée.

**Syntaxe :**  
`bool CBool(exp Expression)`

**Remarques :**  
Si l’expression renvoie une valeur autre que zéro, CBool renvoie la valeur True, sinon elle renvoie False.

**Exemple :**  
`CBool([attrib1] = [attrib2])`  

Retourne True si les attributs ont la même valeur.

---
### <a name="convertfrombase64"></a>ConvertFromBase64
**Description :**  
La fonction ConvertFromBase64 convertit la valeur encodée en base64 en chaîne régulière.

**Syntaxe :**  
`str ConvertFromBase64(str source)` - part du principe que l’encodage utilisé est Unicode  
`str ConvertFromBase64(str source, enum Encoding)`

* source : Chaîne encodée Base64  
* Encodage : Unicode, ASCII, UTF8

**Exemple**  
`ConvertFromBase64("SABlAGwAbABvACAAdwBvAHIAbABkACEA")`  
`ConvertFromBase64("SGVsbG8gd29ybGQh", UTF8)`

Les deux exemples renvoient «*Hello world!* »

---
### <a name="converttobase64"></a>ConvertToBase64
**Description :**  
La fonction ConvertToBase64 convertit une chaîne en chaîne Unicode base64.  
Convertit la valeur d’un tableau d’entiers en sa représentation sous forme de chaîne équivalente encodée avec des chiffres en base 64.

**Syntaxe :**  
`str ConvertToBase64(str source)`

**Exemple :**  
`ConvertToBase64("Hello world!")`  
Renvoie « SABlAGwAbABvACAAdwBvAHIAbABkACEA ».

---
### <a name="converttoutf8hex"></a>ConvertToUTF8Hex
**Description :**  
La fonction ConvertToUTF8Hex convertit une chaîne en valeur hexadécimale encodée UTF8.

**Syntaxe :**  
`str ConvertToUTF8Hex(str source)`

**Remarques :**  
Le format de sortie de cette fonction est utilisé par Azure Active Directory en tant que format d’attribut de nom de domaine.

**Exemple :**  
`ConvertToUTF8Hex("Hello world!")`  
Renvoie 48656C6C6F20776F726C6421.

---
### <a name="count"></a>Count
**Description :**  
La fonction Count renvoie le nombre d’éléments dans un attribut à valeurs multiples.

**Syntaxe :**  
`num Count(mvstr attribute)`

---
### <a name="cstr"></a>CChaîne
**Description :**  
La fonction CStr convertit en un type de données de chaîne.

**Syntaxe :**  
`str CStr(num value)`  
`str CStr(ref value)`  
`str CStr(bool value)`  

* value : Peut être une valeur numérique, un attribut de référence ou une valeur booléenne.

**Exemple :**  
`CStr([dn])`  
Peut renvoyer « cn=Joe,dc=contoso,dc=com ».

---
### <a name="datefromnum"></a>DateFromNum
**Description :**  
La fonction DateFromNum convertit une valeur au format de date AD en un type DateTime.

**Syntaxe :**  
`dt DateFromNum(num value)`

**Exemple :**  
`DateFromNum([lastLogonTimestamp])`  
`DateFromNum(129699324000000000)`  
Renvoie une valeur DateTime représentant 2012-01-01 23:00:00.

---
### <a name="dncomponent"></a>DNComponent
**Description :**  
La fonction DNComponent renvoie la valeur d’un composant de nom de domaine spécifié en partant de la gauche.

**Syntaxe :**  
`str DNComponent(ref dn, num ComponentNumber)`

* dn : attribut de référence à interpréter
* ComponentNumber : Composant du nom de domaine à retourner

**Exemple :**  
`DNComponent(CRef([dn]),1)`  
Si dn est « cn=Joe,ou=… », la fonction renvoie Joe.

---
### <a name="error"></a>Error
**Description :**  
La fonction Error sert à renvoyer une erreur personnalisée.

**Syntaxe :**  
`void Error(str ErrorMessage)`

**Exemple :**  
`IIF(IsPresent([accountName]),[accountName],Error("AccountName is required"))`  
Si l’attribut accountName n’est pas présent, renvoie une erreur sur l’objet.

---
### <a name="formatdatetime"></a>FormatDateTime
**Fonction :**<br> FormatDateTime(source, inputFormat, outputFormat)

**Description :**<br> prend une chaîne de date dans un format et la convertit dans un autre format.

**Paramètres :**<br> 

   | Nom | Requis / Répétition | Type | Notes |
   | --- | --- | --- | --- |
   | **source** |Obligatoire |String |Généralement le nom de l’attribut de l’objet source. |
   | **inputFormat** |Obligatoire |String |Format attendu de la valeur source. Pour les formats pris en charge, consultez [/dotnet/standard/base-types/custom-date-and-time-format-strings](/dotnet/standard/base-types/custom-date-and-time-format-strings). |
   | **outputFormat** |Obligatoire |String |Format de la date de sortie. |

---
### <a name="guid"></a>Guid
**Description :**  
La fonction Guid génère un nouveau GUID aléatoire.

**Syntaxe :**  
`str Guid()`

---
### <a name="iif"></a>IIF
**Description :**  
La fonction IIF renvoie une valeur parmi un ensemble de valeurs possibles en fonction d’une condition spécifiée.

**Syntaxe :**  
`var IIF(exp condition, var valueIfTrue, var valueIfFalse)`

* condition : toute valeur ou expression qui peut être évaluée à true ou false.
* valueIfTrue : La valeur renvoyée si la condition prend la valeur true.
* valueIfFalse : La valeur renvoyée si la condition prend la valeur false.

**Exemple :**  
`IIF([employeeType]="Intern","t-" & [alias],[alias])`  
 Renvoie l’alias d’un utilisateur avec le préfixe « t- » si l’utilisateur est stagiaire. Sinon, l’alias reste inchangé.

---
### <a name="instr"></a>InStr
**Description :**  
La fonction InStr recherche la première occurrence d’une sous-chaîne dans une chaîne.

**Syntaxe :**  

`num InStr(str stringcheck, str stringmatch)`  
`num InStr(str stringcheck, str stringmatch, num start)`  
`num InStr(str stringcheck, str stringmatch, num start, enum compare)`

* stringcheck : chaîne à rechercher
* stringmatch : chaîne à trouver
* start : position de départ pour trouver la sous-chaîne
* compare : vbTextCompare ou vbBinaryCompare

**Remarques :**  
Renvoie la position à laquelle la sous-chaîne a été trouvée, ou 0 si elle est introuvable.

**Exemple :**  
`InStr("The quick brown fox","quick")`  
Prend la valeur 5.

`InStr("repEated","e",3,vbBinaryCompare)`  
Prend la valeur 7.

---
### <a name="isnull"></a>IsNull
**Description :**  
La fonction IsNull renvoie true si l’expression correspond à la valeur Null.

**Syntaxe :**  
`bool IsNull(var Expression)`

**Remarques :**  
Dans le cas d’un attribut, la valeur Null est exprimée par l’absence de ce dernier.

**Exemple :**  
`IsNull([displayName])`  
Renvoie True si l’attribut est absent dans CS ou MV.

---
### <a name="isnullorempty"></a>IsNullOrEmpty
**Description :**  
La fonction IsNullOrEmpty renvoie la valeur true si l’expression a pour valeur Null ou s’il s’agit d’une chaîne vide.

**Syntaxe :**  
`bool IsNullOrEmpty(var Expression)`

**Remarques :**  
Dans le cas d’un attribut, cela donne la valeur True si l’attribut est absent ou est présent mais qu’il s’agit d’une chaîne vide.  
L’inverse de cette fonction est nommé IsPresent.

**Exemple :**  
`IsNullOrEmpty([displayName])`  
Renvoie True si l’attribut est absent dans CS ou MV ou s’il s’agit d’une chaîne vide.

---
### <a name="ispresent"></a>IsPresent
**Description :**  
La fonction IsPresent renvoie true si l’expression correspond à une chaîne qui n’a pas la valeur Null et n’est pas vide.

**Syntaxe :**  
`bool IsPresent(var expression)`

**Remarques :**  
L’inverse de cette fonction est appelé IsNullOrEmpty.

**Exemple :**  
`Switch(IsPresent([directManager]),[directManager], IsPresent([skiplevelManager]),[skiplevelManager], IsPresent([director]),[director])`

---
### <a name="item"></a>Élément
**Description :**  
La fonction Item renvoie un élément à partir d’une chaîne/d’un attribut à valeurs multiples.

**Syntaxe :**  
`var Item(mvstr attribute, num index)`

* attribute : attribut à valeurs multiples
* index : index vers un élément dans la chaîne à valeurs multiples.

**Remarques :**  
la fonction Item est utile si utilisée avec la fonction Contains, car cette dernière renvoie l’index à un élément de l’attribut à valeurs multiples.

Génère une erreur si l’index est hors limites.

**Exemple :**  
`Mid(Item([proxyAddresses],Contains([proxyAddresses], "SMTP:")),6)`  
Renvoie l’adresse de messagerie principale.

---
### <a name="isstring"></a>IsString
**Description :**  
La fonction IsString prend la valeur True si l’expression peut être évaluée en tant que type de chaîne.

**Syntaxe :**  
`bool IsString(var expression)`

**Remarques :**  
Permet de déterminer si CStr() peut parvenir à analyser l’expression.

---
### <a name="join"></a>Join
**Fonction :**<br> Join(séparateur, source1, source2, …)

**Description :**<br> Join() est similaire à Append(), mais elle peut combiner plusieurs valeurs de chaîne **sources** dans une même chaîne et chaque valeur sera séparée par une chaîne de **séparation**.

Si l’une des valeurs sources est un attribut à valeurs multiples, toutes les valeurs de cet attribut seront jointes, séparées par la valeur de séparation.

**Paramètres :**<br> 

   | Nom | Requis / Répétition | Type | Notes |
   | --- | --- | --- | --- |
   | **separator** |Obligatoire |String |Chaîne utilisée pour séparer les valeurs sources quand elles sont concaténées en une seule chaîne. Peut être "" si aucun séparateur n’est requis. |
   | **source1  … sourceN** |Requis, nombre de fois variable |String |Valeurs de chaîne à joindre ensemble. |

---
### <a name="left"></a>Gauche
**Description :**  
La fonction Left renvoie un nombre spécifié de caractères en partant de la gauche d’une chaîne.

**Syntaxe :**  
`str Left(str string, num NumChars)`

* string : chaîne à partir de laquelle les caractères sont renvoyés
* NumChars : nombre identifiant le nombre de caractères à retourner du début (à gauche) de la chaîne

**Remarques :**  
Chaîne contenant les numChars premiers caractères de la chaîne :

* Si numChars = 0, retourne une chaîne vide.
* Si numChars < 0, retourne une chaîne d’entrée.
* Si la chaîne est null, retourne une chaîne vide.

Si la chaîne contient moins de caractères que le nombre spécifié dans numChars, une chaîne identique à la chaîne (c’est-à-dire, contenant tous les caractères du paramètre 1) est renvoyée.

**Exemple :**  
`Left("John Doe", 3)`  
Retourne `Joh`.

---
### <a name="mid"></a>ExtracChaîne
**Fonction :**<br> Mid(source, début, longueur)

**Description :**<br> retourne une sous-chaîne de la valeur source. Une sous-chaîne est une chaîne qui ne contient que certains des caractères de la chaîne source.

**Paramètres :**<br> 

   | Nom | Requis / Répétition | Type | Notes |
   | --- | --- | --- | --- |
   | **source** |Obligatoire |String |Généralement le nom de l’attribut. |
   | **start** |Obligatoire |entier |Index dans la chaîne **source** où la sous-chaîne doit commencer. Le premier caractère dans la chaîne aura l’index 1, le deuxième caractère aura l’index 2, et ainsi de suite. |
   | **length** |Obligatoire |entier |Longueur de la sous-chaîne. Si la longueur se termine à l’extérieur de la chaîne **source**, la fonction retourne la sous-chaîne de l’index **start** jusqu’à la fin de l’index **source**. |

---
### <a name="normalizediacritics"></a>NormalizeDiacritics
**Fonction :**<br> NormalizeDiacritics(source)

**Description :**<br> Nécessite un argument de chaîne. Retourne la chaîne, où les caractères diacritiques ont été remplacés par leurs équivalents non diacritiques. Généralement utilisé pour convertir les prénoms et les noms contenant des caractères diacritiques (accents) en valeurs autorisées pouvant être utilisées dans différents identificateurs d’utilisateurs, tels que les noms d’utilisateurs principaux, les noms de compte SAM et les adresses e-mail.

**Paramètres :**<br> 

   | Nom | Requis / Répétition | Type | Notes |
   | --- | --- | --- | --- |
   | **source** |Obligatoire |String | Généralement un attribut de nom ou de prénom. |

---
### <a name="not"></a>Not
**Fonction :**<br> Not(source)

**Description :**<br> inverse la valeur booléenne de la **source**. Si la valeur **source** est « *True* », cette fonction retourne «*False* ». Sinon, elle retourne «*True*».

**Paramètres :**<br> 

   | Nom | Requis / Répétition | Type | Notes |
   | --- | --- | --- | --- |
   | **source** |Obligatoire |Chaîne de type Boolean |Les valeurs **sources** attendues sont « True » ou « False ». |

---
### <a name="removeduplicates"></a>RemoveDuplicates
**Description :**  
La fonction RemoveDuplicates prend une chaîne à valeurs multiples et vérifie que chaque valeur est unique.

**Syntaxe :**  
`mvstr RemoveDuplicates(mvstr attribute)`

**Exemple :**  
`RemoveDuplicates([proxyAddresses])`  
Renvoie un attribut proxyAddress expurgé duquel toutes les valeurs en double ont été supprimées.

---
### <a name="replace"></a>Replace
**Fonction :**<br> Remplacer (source, oldValue, regexPattern, regexGroupName, replacementValue, replacementAttributeName, template)

**Description :**<br>
Remplace les valeurs dans une chaîne. Elle fonctionne différemment selon les paramètres fournis :

* Quand **oldValue** et **replacementValue** sont fournis :
  
  * Remplace toutes les occurrences d’**oldValue** dans la **source** par  **replacementValue**.
* Quand **oldValue** et **template** sont fournis :
  
  * Remplace toutes les occurrences d’**oldValue** dans le **template** par la valeur **source**.
* Quand **regexPattern** et **replacementValue** sont fournis :

  * La fonction applique **regexPattern** à la chaîne **source** et vous pouvez utiliser les noms de groupes regex pour construire la chaîne pour **replacementValue**.
* Quand **regexPattern**, **regexGroupName** et **replacementValue** sont fournis :
  
  * La fonction applique **regexPattern** à la chaîne **source** et remplace toutes les valeurs correspondant à **regexGroupName** par **replacementValue**.
* Quand **regexPattern**, **regexGroupName** et **replacementAttributeName** sont fournis :
  
  * Si **source** n’a pas de valeur, **source** est retourné
  * Si **source** a une valeur, la fonction applique **regexPattern** à la chaîne **source** et remplace toutes les valeurs correspondant à **regexGroupName** par la valeur associée à **replacementAttributeName**.

**Paramètres :**<br> 

   | Nom | Requis / Répétition | Type | Notes |
   | --- | --- | --- | --- |
   | **source** |Obligatoire |String |Généralement, nom de l’attribut de l’objet **source**. |
   | **oldValue** |Facultatif |String |Valeur à remplacer dans **source** ou **template**. |
   | **regexPattern** |Facultatif |String |Modèle d’expression régulière pour la valeur à remplacer dans **source**. Ou, quand **replacementPropertyName** est utilisé, modèle pour extraire la valeur de **replacementPropertyName**. |
   | **regexGroupName** |Facultatif |String |Nom du groupe à l’intérieur de **regexPattern**. Nous extrayons la valeur de ce groupe en tant que **replacementValue** à partir de **replacementPropertyName** uniquement quand **replacementPropertyName** est utilisé. |
   | **replacementValue** |Facultatif |String |Nouvelle valeur par laquelle remplacer l’ancienne. |
   | **replacementAttributeName** |Facultatif |String |Nom de l’attribut à utiliser pour la valeur de remplacement. |
   | **template** |Facultatif |String |Lorsque la valeur **template** est fournie, nous recherchons la valeur **oldValue** dans le modèle et la remplaçons par la valeur **source**. |

---
### <a name="selectuniquevalue"></a>SelectUniqueValue
**Fonction :**<br> SelectUniqueValue(uniqueValueRule1, uniqueValueRule2, uniqueValueRule3, …)

**Description :**<br> Nécessite un minimum de deux arguments, qui sont définis à l’aide d’expressions de règles de génération de valeur unique. La fonction évalue chaque règle, puis vérifie la valeur générée pour l’unicité dans le répertoire/application cible. La première valeur unique trouvée est celle retournée. Si toutes les valeurs existent déjà dans la cible, l’entrée sera déposée et le motif de cette action consigné dans les journaux d’audit. Il n’existe aucune limite supérieure au nombre d’arguments qui peuvent être fournis.

> [!NOTE]
> - Il s’agit d’une fonction de niveau supérieur, vous ne pouvez donc pas l’imbriquer.
> - Cette fonction ne peut pas s’appliquer à des attributs qui ont une priorité de correspondance.  
> - Cette fonction est uniquement destinée à être utilisée pour les créations d’entrées. Lorsque vous l’utilisez avec un attribut, définissez la propriété **Appliquer le mappage** sur **Uniquement durant la création d’objet**.
> - Cette fonction est actuellement prise en charge uniquement pour le « Provisionnement d’utilisateurs de Workday vers Active Directory ». Elle ne peut pas être utilisée avec d’autres applications de provisionnement. 


**Paramètres :**<br> 

   | Nom | Requis / Répétition | Type | Notes |
   | --- | --- | --- | --- |
   | **uniqueValueRule1  … uniqueValueRuleN** |Au moins 2 requis, aucune limite supérieure |String | Liste des règles de génération de valeur unique à évaluer. |


---
### <a name="singleapproleassignment"></a>SingleAppRoleAssignment
**Fonction :**<br> SingleAppRoleAssignment([appRoleAssignments])

**Description :**<br> Retourne un appRoleAssignment unique parmi la liste de tous les appRoleAssignments affectés à un utilisateur pour une application donnée. Cette fonction est nécessaire pour convertir l’objet appRoleAssignments en une chaîne de nom de rôle unique. Notez que la bonne pratique consiste à s’assurer qu’un seul appRoleAssignment est attribué à un seul utilisateur à la fois, et si plusieurs rôles sont attribués, la chaîne de rôle retournée ne doit pas être prévisible. 

**Paramètres :**<br> 

  | Nom | Requis / Répétition | Type | Notes |
  |--- | --- | --- | --- |
  | **[appRoleAssignments]** |Obligatoire |String |Objet **[appRoleAssignments]** . |

---
### <a name="split"></a>Split
**Fonction :**<br> Split (source, délimiteur)

**Description :**<br> Fractionne une chaîne en un tableau à plusieurs valeurs, en utilisant le caractère délimiteur spécifié.

**Paramètres :**<br> 

   | Nom | Requis / Répétition | Type | Notes |
   | --- | --- | --- | --- |
   | **source** |Obligatoire |String |**source** à mettre à jour. |
   | **délimiteur** |Obligatoire |String |Spécifie le caractère qui sera utilisé pour fractionner la chaîne (exemple : « , ») |

---
### <a name="stringfromsid"></a>StringFromSid
**Description :**  
La fonction StringFromSid convertit en chaîne un tableau d’octets contenant un identificateur de sécurité.

**Syntaxe :**  
`str StringFromSid(bin ObjectSID)`  

---
### <a name="stripspaces"></a>StripSpaces
**Fonction :**<br> StripSpaces(source)

**Description :**<br> supprime tous les caractères d’espacement (" ") de la chaîne source.

**Paramètres :**<br> 

   | Nom | Requis / Répétition | Type | Notes |
   | --- | --- | --- | --- |
   | **source** |Obligatoire |String |**source** à mettre à jour. |

---
### <a name="switch"></a>Commutateur
**Fonction :**<br> Switch(source, defaultValue, key1, value1, key2, value2, …)

**Description :**<br> quand la valeur **source** correspond à une **clé**, retourne la **valeur** de cette **clé**. Si la valeur **source** ne correspond à aucune clé, retourne **defaultValue**.  Les paramètres **key** et **value** doivent toujours être fournis par paires. La fonction attend toujours un nombre pair de paramètres.

**Paramètres :**<br> 

   | Nom | Requis / Répétition | Type | Notes |
   | --- | --- | --- | --- |
   | **source** |Obligatoire |String |Valeur **source** à vérifier. |
   | **defaultValue** |Facultatif |String |Valeur par défaut à utiliser quand la source ne correspond à aucune clé. Peut être une chaîne vide (""). |
   | **key** |Obligatoire |String |**Key** avec laquelle comparer la valeur **source**. |
   | **value** |Obligatoire |String |Valeur de remplacement pour la **source** correspondant à la clé. |

---
### <a name="tolower"></a>ToLower
**Fonction :**<br> ToLower(source, culture)

**Description :**<br> prend une valeur de la chaîne *source* et la convertit en minuscules à l’aide des règles de culture spécifiées. Si aucune information de *culture* n'est spécifiée, une culture invariante est utilisée.

**Paramètres :**<br> 

   | Nom | Requis / Répétition | Type | Notes |
   | --- | --- | --- | --- |
   | **source** |Obligatoire |String |Généralement le nom de l’attribut de l’objet source |
   | **culture** |Facultatif |String |Le format du nom de culture basé sur RFC 4646 est *languagecode2-country/regioncode2*, où *languagecode2* correspond au code de langue à deux lettres et *country/regioncode2* au code de sous-culture à deux lettres, par exemple, ja-JP pour le japonais (Japon) et en-US pour l’anglais (États-Unis). Si un code de langue à deux lettres n'est pas disponible, un code à trois lettres dérivé de la norme ISO 639-2 est utilisé.|

---

### <a name="toupper"></a>ToUpper
**Fonction :**<br> ToUpper(source, culture)

**Description :**<br> prend une valeur de la chaîne *source* et la convertit en majuscule à l’aide des règles de culture spécifiées. Si aucune information de *culture* n'est spécifiée, une culture invariante est utilisée.

**Paramètres :**<br> 

  | Nom | Requis / Répétition | Type | Notes |
  | --- | --- | --- | --- |
  | **source** |Obligatoire |String |Généralement le nom de l’attribut de l’objet source. |
  | **culture** |Facultatif |String |Le format du nom de culture basé sur RFC 4646 est *languagecode2-country/regioncode2*, où *languagecode2* correspond au code de langue à deux lettres et *country/regioncode2* au code de sous-culture à deux lettres, par exemple, ja-JP pour le japonais (Japon) et en-US pour l’anglais (États-Unis). Si un code de langue à deux lettres n'est pas disponible, un code à trois lettres dérivé de la norme ISO 639-2 est utilisé.|

---

### <a name="trim"></a>SupprEspace
**Description :**  
La fonction Trim supprime les espaces blancs situés au début et à la fin d’une chaîne.

**Syntaxe :**  
`str Trim(str value)`  

**Exemple :**  
`Trim(" Test ")`  
Renvoie « test ».

`Trim([proxyAddresses])`  
Supprime les espaces blancs de début et de fin pour chaque valeur contenue dans l’attribut proxyAddress.

---
### <a name="word"></a>Word
**Description :**  
La fonction Word retourne un mot contenu dans une chaîne, en fonction des paramètres qui décrivent les délimiteurs à utiliser et le nombre de mots à retourner.

**Syntaxe :**  
`str Word(str string, num WordNumber, str delimiters)`

* string : chaîne à partir de laquelle le mot est renvoyé.
* WordNumber : nombre identifiant le nombre de mots à renvoyer.
* delimiters : chaîne représentant le ou les délimiteur(s) à utiliser pour identifier les mots

**Remarques :**  
Chaque chaîne de caractères contenue dans la chaîne séparée par l’un des caractères figurant dans delimiters est identifiée en tant que mot :

* Si number < 1, retourne une chaîne vide.
* Si string a la valeur null, renvoie une chaîne vide.

Si la chaîne contient moins de mots ou ne contient pas les mots identifiés par les séparateurs, une chaîne vide est renvoyée.

**Exemple :**  
`Word("The quick brown fox",3," ")`  
Retourne « brown ».

`Word("This,string!has&many separators",3,",!&#")`  
Retourne « has ».

## <a name="examples"></a>Exemples
### <a name="strip-known-domain-name"></a>Supprimer un nom de domaine connu
Vous devez supprimer un nom de domaine connu de l’adresse de messagerie d’un utilisateur pour obtenir un nom d’utilisateur. <br>
Par exemple, si le domaine est « contoso.com », vous pouvez utiliser l’expression suivante :

**Expression :** <br>
`Replace([mail], "@contoso.com", , ,"", ,)`

**Exemple d’entrée/sortie :** <br>

* **ENTRÉE** (e-mail) : "john.doe@contoso.com"
* **SORTIE** : « john.doe »

### <a name="append-constant-suffix-to-user-name"></a>Ajouter un suffixe de constante à un nom d’utilisateur
Si vous utilisez un Sandbox Salesforce, vous devrez peut-être ajouter un suffixe supplémentaire à tous les noms d’utilisateurs avant de les synchroniser.

**Expression :** <br>
`Append([userPrincipalName], ".test")`

**Exemple d’entrée/sortie :** <br>

* **ENTRÉE** : (userPrincipalName) : "John.Doe@contoso.com"
* **SORTIE**:  "John.Doe@contoso.com.test"

### <a name="generate-user-alias-by-concatenating-parts-of-first-and-last-name"></a>Générer des alias d’utilisateurs en concaténant des parties du prénom et du nom
Vous devez générer un alias d’utilisateur en prenant les trois premières lettres du prénom de l’utilisateur et les cinq premières lettres de son nom de famille.

**Expression :** <br>
`Append(Mid([givenName], 1, 3), Mid([surname], 1, 5))`

**Exemple d’entrée/sortie :** <br>

* **ENTRÉE** (givenName) : "John"
* **ENTRÉE** (surname) : "Doe"
* **SORTIE** :  "JohDoe"

### <a name="remove-diacritics-from-a-string"></a>Supprimer les signes diacritiques d’une chaîne
Vous devez remplacer les caractères accentués par leurs équivalents non accentués.

**Expression :** <br>
NormalizeDiacritics([givenName])

**Exemple d’entrée/sortie :** <br>

* **ENTRÉE** (givenName) : "Zoë"
* **SORTIE** :  "Zoe"

### <a name="split-a-string-into-a-multi-valued-array"></a>Fractionner une chaîne en un tableau à valeurs multiples
Vous devez prendre une liste de chaînes délimitées par des virgules, et la fractionner en un tableau pouvant être raccordé à un attribut à valeurs multiples comme l’attribut PermissionSets de Salesforce. Dans cet exemple, une liste des jeux d’autorisations a été remplie dans extensionAttribute5 dans Azure AD.

**Expression :** <br>
Split([extensionAttribute5], ",")

**Exemple d’entrée/sortie :** <br>

* **INPUT** (extensionAttribute5): "PermissionSetOne, PermissionSetTwo"
* **SORTIE** :  ["PermissionSetOne", "PermissionSetTwo"]

### <a name="output-date-as-a-string-in-a-certain-format"></a>Sortir une date sous la forme d’une chaîne dans un certain format
Vous souhaitez envoyer des dates à une application SaaS dans un format donné. <br>
Par exemple, vous souhaitez mettre en forme des dates pour ServiceNow.

**Expression :** <br>

`FormatDateTime([extensionAttribute1], "yyyyMMddHHmmss.fZ", "yyyy-MM-dd")`

**Exemple d’entrée/sortie :**

* **ENTRÉE** (extensionAttribute1) : "20150123105347.1Z"
* **SORTIE** :  "2015-01-23"

### <a name="replace-a-value-based-on-predefined-set-of-options"></a>Remplacer une valeur en fonction d’un ensemble d’options prédéfini

Vous devez définir le fuseau horaire de l’utilisateur en fonction du code d’état stocké dans Azure AD. <br>
Si le code d’état ne correspond à aucune des options prédéfinies, utilisez la valeur par défaut « Australia/Sydney ».

**Expression :** <br>
`Switch([state], "Australia/Sydney", "NSW", "Australia/Sydney","QLD", "Australia/Brisbane", "SA", "Australia/Adelaide")`

**Exemple d’entrée/sortie :**

* **ENTRÉE** (état) : "QLD"
* **SORTIE** : "Australia/Brisbane"

### <a name="replace-characters-using-a-regular-expression"></a>Remplacer des caractères à l’aide d’une expression régulière
Vous devez rechercher des caractères correspondant à une valeur d’expression régulière et les supprimer.

**Expression :** <br>

Replace([mailNickname], , "[a-zA-Z_]*", , "", , )

**Exemple d’entrée/sortie :**

* **ENTRÉE** (mailNickname: "john_doe72"
* **SORTIE** : "72"

### <a name="convert-generated-userprincipalname-upn-value-to-lower-case"></a>Convertir la valeur userPrincipalName (UPN) générée en minuscules
Dans l’exemple ci-dessous, la valeur UPN est générée en concaténant les champs sources PreferredFirstName et PreferredLastName, et la fonction ToLower agit sur la chaîne générée pour convertir tous les caractères en minuscules. 

`ToLower(Join("@", NormalizeDiacritics(StripSpaces(Join(".",  [PreferredFirstName], [PreferredLastName]))), "contoso.com"))`

**Exemple d’entrée/sortie :**

* **ENTRÉE** (PreferredFirstName) : "John"
* **ENTRÉE** (PreferredLastName) : "Smith"
* **SORTIE** : "john.smith@contoso.com"

### <a name="generate-unique-value-for-userprincipalname-upn-attribute"></a>Générer une valeur unique pour l’attribut userPrincipalName (UPN)
En fonction du prénom, du deuxième prénom et du nom de famille de l’utilisateur, vous devez générer une valeur pour l’attribut UPN et vérifier son caractère unique dans le répertoire AD cible avant d’attribuer la valeur à l’attribut UPN.

**Expression :** <br>

```ad-attr-mapping-expr
    SelectUniqueValue( 
        Join("@", NormalizeDiacritics(StripSpaces(Join(".",  [PreferredFirstName], [PreferredLastName]))), "contoso.com"), 
        Join("@", NormalizeDiacritics(StripSpaces(Join(".",  Mid([PreferredFirstName], 1, 1), [PreferredLastName]))), "contoso.com"),
        Join("@", NormalizeDiacritics(StripSpaces(Join(".",  Mid([PreferredFirstName], 1, 2), [PreferredLastName]))), "contoso.com")
    )
```

**Exemple d’entrée/sortie :**

* **ENTRÉE** (PreferredFirstName) : "John"
* **ENTRÉE** (PreferredLastName) : "Smith"
* **SORTIE** : « John.Smith@contoso.com » si la valeur UPN de John.Smith@contoso.com n’existe pas déjà dans le répertoire
* **SORTIE** : « J.Smith@contoso.com » si la valeur UPN de John.Smith@contoso.com existe déjà dans le répertoire
* **SORTIE** : « Jo.Smith@contoso.com » si les deux valeurs UPN précédentes existent déjà dans le répertoire


## <a name="next-steps"></a>Étapes suivantes 

- [Présentation du provisionnement](what-is-provisioning.md)
- [Présentation du provisionnement cloud Azure AD Connect](what-is-cloud-provisioning.md)