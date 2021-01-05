---
title: 'Synchronisation d’Azure AD Connect : Informations de référence sur les fonctions | Microsoft Docs'
description: Référence d’expressions d’approvisionnement déclaratif dans Azure AD Connect Sync.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 4f525ca0-be0e-4a2e-8da1-09b6b567ed5f
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 07/12/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: c6c8be064ade8182355c320e948b3b60b846033d
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96348056"
---
# <a name="azure-ad-connect-sync-functions-reference"></a>Synchronisation d’Azure AD Connect : Référence des fonctions
Dans Azure AD Connect, les fonctions servent à manipuler une valeur d’attribut pendant la synchronisation.  
La syntaxe des fonctions s’exprime selon le format suivant :  
`<output type> FunctionName(<input type> <position name>, ..)`

Si une fonction est surchargée et accepte plusieurs syntaxes, toutes les syntaxes valides sont répertoriées.  
Les fonctions sont fortement typées et vérifient que le type passé correspond au type documenté.  
Une erreur est renvoyée si le type ne correspond pas.

Les types s’expriment avec la syntaxe suivante :

* **bin** : binaire
* **bool** : booléen
* **dt** : date/heure UTC
* **enum** : énumération des constantes connues
* **exp** : expression, qui est censée correspondre à une valeur booléenne
* **mvbin** : binaire à valeurs multiples
* **mvstr** : chaîne à valeurs multiples
* **mvref** : référence à valeurs multiples
* **num** : numérique
* **ref** : référence
* **str** : chaîne
* **var** : variante de (quasiment) tout autre type
* **void** : ne retourne aucune valeur

Les fonctions ayant pour type **mvbin**, **mvstr** et **mvref** ne peuvent fonctionner que sur les attributs à valeurs multiples. Les fonctions avec **bin**, **str** et **ref** fonctionnent sur des attributs à valeur unique et à valeurs multiples.

## <a name="functions-reference"></a>Référence des fonctions

* **Certificate**
  * [CertExtensionOids](#certextensionoids)
  * [CertFormat](#certformat)
  * [CertFriendlyName](#certfriendlyname)
  * [CertHashString](#certhashstring)
  * [CertIssuer](#certissuer)
  * [CertIssuerDN](#certissuerdn)
  * [CertIssuerOid](#certissueroid)
  * [CertKeyAlgorithm](#certkeyalgorithm)
  * [CertKeyAlgorithmParams](#certkeyalgorithmparams)
  * [CertNameInfo](#certnameinfo)
  * [CertNotAfter](#certnotafter)
  * [CertNotBefore](#certnotbefore)
  * [CertPublicKeyOid](#certpublickeyoid)
  * [CertPublicKeyParametersOid](#certpublickeyparametersoid)
  * [CertSerialNumber](#certserialnumber)
  * [CertSignatureAlgorithmOid](#certsignaturealgorithmoid)
  * [CertSubject](#certsubject)
  * [CertSubjectNameDN](#certsubjectnamedn)
  * [CertSubjectNameOid](#certsubjectnameoid)
  * [CertThumbprint](#certthumbprint)
  * [CertVersion](#certversion)
  * [IsCert](#iscert)
* **Conversion**
  * [CBool](#cbool)
  * [CDate](#cdate)
  * [CGuid](#cguid)
  * [ConvertFromBase64](#convertfrombase64)
  * [ConvertToBase64](#converttobase64)
  * [ConvertFromUTF8Hex](#convertfromutf8hex)
  * [ConvertToUTF8Hex](#converttoutf8hex)
  * [CNum](#cnum)
  * [CRef](#cref)
  * [CStr](#cstr)
  * [StringFromGuid](#stringfromguid)
  * [StringFromSid](#stringfromsid)
* **Date / Heure**
  * [DateAdd](#dateadd)
  * [DateFromNum](#datefromnum)
  * [FormatDateTime](#formatdatetime)
  * [Now](#now)
  * [NumFromDate](#numfromdate)
* **Directory**
  * [DNComponent](#dncomponent)
  * [DNComponentRev](#dncomponentrev)
  * [EscapeDNComponent](#escapedncomponent)
* **Evaluation**
  * [IsBitSet](#isbitset)
  * [IsDate](#isdate)
  * [IsEmpty](#isempty)
  * [IsGuid](#isguid)
  * [IsNull](#isnull)
  * [IsNullOrEmpty](#isnullorempty)
  * [IsNumeric](#isnumeric)
  * [IsPresent](#ispresent)
  * [IsString](#isstring)
* **Mathématique**
  * [BitAnd](#bitand)
  * [BitOr](#bitor)
  * [RandomNum](#randomnum)
* **Valeur multiple**
  * [Contains](#contains)
  * [Count](#count)
  * [Item](#item)
  * [ItemOrNull](#itemornull)
  * [Join](#join)
  * [RemoveDuplicates](#removeduplicates)
  * [Split](#split)
* **Flux de programme**
  * [Error](#error)
  * [IIF](#iif)
  * [Select](#select)
  * [Switch](#switch)
  * [Where](#where)
  * [With](#with)
* **Text**
  * [GUID](#guid)
  * [InStr](#instr)
  * [InStrRev](#instrrev)
  * [LCase](#lcase)
  * [Left](#left)
  * [Len](#len)
  * [LTrim](#ltrim)
  * [Mid](#mid)
  * [PadLeft](#padleft)
  * [PadRight](#padright)
  * [PCase](#pcase)
  * [Replace](#replace)
  * [ReplaceChars](#replacechars)
  * [Right](#right)
  * [RTrim](#rtrim)
  * [Trim](#trim)
  * [UCase](#ucase)
  * [Word](#word)

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
`BitAnd(&HF, &HF7)`  
Renvoie 7, car les valeurs hexadécimales « F » ET « F7 » donnent cette valeur.

---
### <a name="bitor"></a>BitOr
**Description :**  
La fonction BitOr définit des bits spécifiés sur une valeur.

**Syntaxe :**  
`num BitOr(num value1, num value2)`

* value1, value2 : valeurs numériques qui doivent être liées par OR

**Remarques :**  
Cette fonction convertit les deux paramètres en la représentation binaire et définit un bit sur 1 si l’un des bits ou les deux bits correspondants dans le masque et l’indicateur ont pour valeur 1, ou sur 0 si les deux bits correspondants ont la valeur 0. En d’autres termes, elle renvoie 1 dans tous les cas, sauf si les bits correspondants de ces deux paramètres ont pour valeur 0.

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
### <a name="cdate"></a>CDate
**Description :**  
La fonction CDate renvoie une valeur DateTime UTC à partir d’une chaîne. DateTime n’est pas un type d’attribut natif dans Sync, mais il est utilisé par certaines fonctions.

**Syntaxe :**  
`dt CDate(str value)`

* Valeur : Chaîne comportant une date, une heure, et éventuellement, un fuseau horaire

**Remarques :**  
La chaîne renvoyée est toujours au format UTC.

**Exemple :**  
`CDate([employeeStartTime])`  
Renvoie une valeur DateTime à partir de l’heure de début de l’employé.

`CDate("2013-01-10 4:00 PM -8")`  
Renvoie une valeur DateTime représentant « 2013-01-11 12:00 AM ».


---
### <a name="certextensionoids"></a>CertExtensionOids
**Description :**  
Retourne les valeurs OID de toutes les extensions critiques d’un objet certificat.

**Syntaxe :**  
`mvstr CertExtensionOids(binary certificateRawData)`  
*   certificateRawData : Représentation en tableau d’octets d’un certificat X.509.Byte array representation of an X.509 certificate. Le tableau d’octets peut contenir des données X.509 codées en binaire (DER) ou en Base64.

---
### <a name="certformat"></a>CertFormat
**Description :**  
Retourne le nom du format de ce certificat X.509v3.

**Syntaxe :**  
`str CertFormat(binary certificateRawData)`  
*   certificateRawData : Représentation en tableau d’octets d’un certificat X.509.Byte array representation of an X.509 certificate. Le tableau d’octets peut contenir des données X.509 codées en binaire (DER) ou en Base64.

---
### <a name="certfriendlyname"></a>CertFriendlyName
**Description :**  
Retourne l’alias associé à un certificat.

**Syntaxe :**  
`str CertFriendlyName(binary certificateRawData)`  
*   certificateRawData : Représentation en tableau d’octets d’un certificat X.509.Byte array representation of an X.509 certificate. Le tableau d’octets peut contenir des données X.509 codées en binaire (DER) ou en Base64.

---
### <a name="certhashstring"></a>CertHashString
**Description :**  
Retourne la valeur de hachage SHA1 du certificat x.509v.3 sous forme de chaîne hexadécimale.

**Syntaxe :**  
`str CertHashString(binary certificateRawData)`  
*   certificateRawData : Représentation en tableau d’octets d’un certificat X.509.Byte array representation of an X.509 certificate. Le tableau d’octets peut contenir des données X.509 codées en binaire (DER) ou en Base64.

---
### <a name="certissuer"></a>CertIssuer
**Description :**  
Retourne le nom de l’autorité de certification qui a émis le certificat x.509v.3.

**Syntaxe :**  
`str CertIssuer(binary certificateRawData)`  
*   certificateRawData : Représentation en tableau d’octets d’un certificat X.509.Byte array representation of an X.509 certificate. Le tableau d’octets peut contenir des données X.509 codées en binaire (DER) ou en Base64.

---
### <a name="certissuerdn"></a>CertIssuerDN
**Description :**  
Retourne le nom unique de l’émetteur de certificat.

**Syntaxe :**  
`str CertIssuerDN(binary certificateRawData)`  
*   certificateRawData : Représentation en tableau d’octets d’un certificat X.509.Byte array representation of an X.509 certificate. Le tableau d’octets peut contenir des données X.509 codées en binaire (DER) ou en Base64.

---
### <a name="certissueroid"></a>CertIssuerOid
**Description :**  
Retourne l’OID de l’émetteur de certificat.

**Syntaxe :**  
`str CertIssuerOid(binary certificateRawData)`  
*   certificateRawData : Représentation en tableau d’octets d’un certificat X.509.Byte array representation of an X.509 certificate. Le tableau d’octets peut contenir des données X.509 codées en binaire (DER) ou en Base64.

---
### <a name="certkeyalgorithm"></a>CertKeyAlgorithm
**Description :**  
Retourne les informations d’algorithme de clé du certificat x.509v.3 sous forme de chaîne.

**Syntaxe :**  
`str CertKeyAlgorithm(binary certificateRawData)`  
*   certificateRawData : Représentation en tableau d’octets d’un certificat X.509.Byte array representation of an X.509 certificate. Le tableau d’octets peut contenir des données X.509 codées en binaire (DER) ou en Base64.

---
### <a name="certkeyalgorithmparams"></a>CertKeyAlgorithmParams
**Description :**  
Retourne les paramètres d’algorithme de clé du certificat x.509v.3 sous forme de chaîne hexadécimale.

**Syntaxe :**  
`str CertKeyAlgorithm(binary certificateRawData)`  
*   certificateRawData : Représentation en tableau d’octets d’un certificat X.509.Byte array representation of an X.509 certificate. Le tableau d’octets peut contenir des données X.509 codées en binaire (DER) ou en Base64.

---
### <a name="certnameinfo"></a>CertNameInfo
**Description :**  
Retourne le nom du sujet et de l’émetteur d’un certificat.

**Syntaxe :**  
`str CertNameInfo(binary certificateRawData, str x509NameType, bool includesIssuerName)`  
*   certificateRawData : Représentation en tableau d’octets d’un certificat X.509.Byte array representation of an X.509 certificate. Le tableau d’octets peut contenir des données X.509 codées en binaire (DER) ou en Base64.
*   X509NameType : Valeur X509NameType du sujet.
*   includesIssuerName : true pour inclure le nom de l’émetteur ; sinon, false.

---
### <a name="certnotafter"></a>CertNotAfter
**Description :**  
Retourne la date en heure locale après laquelle un certificat n’est plus valide.

**Syntaxe :**  
`dt CertNotAfter(binary certificateRawData)`  
*   certificateRawData : Représentation en tableau d’octets d’un certificat X.509.Byte array representation of an X.509 certificate. Le tableau d’octets peut contenir des données X.509 codées en binaire (DER) ou en Base64.

---
### <a name="certnotbefore"></a>CertNotBefore
**Description :**  
Retourne la date en heure locale à partir de laquelle un certificat devient valide.

**Syntaxe :**  
`dt CertNotBefore(binary certificateRawData)`  
*   certificateRawData : Représentation en tableau d’octets d’un certificat X.509.Byte array representation of an X.509 certificate. Le tableau d’octets peut contenir des données X.509 codées en binaire (DER) ou en Base64.

---
### <a name="certpublickeyoid"></a>CertPublicKeyOid
**Description :**  
Retourne l’OID de la clé publique du certificat x.509v.3.

**Syntaxe :**  
`str CertKeyAlgorithm(binary certificateRawData)`  
*   certificateRawData : Représentation en tableau d’octets d’un certificat X.509.Byte array representation of an X.509 certificate. Le tableau d’octets peut contenir des données X.509 codées en binaire (DER) ou en Base64.

---
### <a name="certpublickeyparametersoid"></a>CertPublicKeyParametersOid
**Description :**  
Retourne l’OID des paramètres de clé publique du certificat x.509v.3.

**Syntaxe :**  
`str CertPublicKeyParametersOid(binary certificateRawData)`  
*   certificateRawData : Représentation en tableau d’octets d’un certificat X.509.Byte array representation of an X.509 certificate. Le tableau d’octets peut contenir des données X.509 codées en binaire (DER) ou en Base64.

---
### <a name="certserialnumber"></a>CertSerialNumber
**Description :**  
Retourne le numéro de série du certificat X.509v3.

**Syntaxe :**  
`str CertSerialNumber(binary certificateRawData)`  
*   certificateRawData : Représentation en tableau d’octets d’un certificat X.509.Byte array representation of an X.509 certificate. Le tableau d’octets peut contenir des données X.509 codées en binaire (DER) ou en Base64.

---
### <a name="certsignaturealgorithmoid"></a>CertSignatureAlgorithmOid
**Description :**  
Retourne l’OID de l’algorithme utilisé pour créer la signature d’un certificat.

**Syntaxe :**  
`str CertSignatureAlgorithmOid(binary certificateRawData)`  
*   certificateRawData : Représentation en tableau d’octets d’un certificat X.509.Byte array representation of an X.509 certificate. Le tableau d’octets peut contenir des données X.509 codées en binaire (DER) ou en Base64.

---
### <a name="certsubject"></a>CertSubject
**Description :**  
Obtient le nom unique du sujet du certificat.

**Syntaxe :**  
`str CertSubject(binary certificateRawData)`  
*   certificateRawData : Représentation en tableau d’octets d’un certificat X.509.Byte array representation of an X.509 certificate. Le tableau d’octets peut contenir des données X.509 codées en binaire (DER) ou en Base64.

---
### <a name="certsubjectnamedn"></a>CertSubjectNameDN
**Description :**  
Retourne le nom unique du sujet du certificat.

**Syntaxe :**  
`str CertSubjectNameDN(binary certificateRawData)`  
*   certificateRawData : Représentation en tableau d’octets d’un certificat X.509.Byte array representation of an X.509 certificate. Le tableau d’octets peut contenir des données X.509 codées en binaire (DER) ou en Base64.

---
### <a name="certsubjectnameoid"></a>CertSubjectNameOid
**Description :**  
Retourne l’OID du nom du sujet d’un certificat.

**Syntaxe :**  
`str CertSubjectNameOid(binary certificateRawData)`  
*   certificateRawData : Représentation en tableau d’octets d’un certificat X.509.Byte array representation of an X.509 certificate. Le tableau d’octets peut contenir des données X.509 codées en binaire (DER) ou en Base64.

---
### <a name="certthumbprint"></a>CertThumbprint
**Description :**  
Retourne l’empreinte d’un certificat.

**Syntaxe :**  
`str CertThumbprint(binary certificateRawData)`  
*   certificateRawData : Représentation en tableau d’octets d’un certificat X.509.Byte array representation of an X.509 certificate. Le tableau d’octets peut contenir des données X.509 codées en binaire (DER) ou en Base64.

---
### <a name="certversion"></a>CertVersion
**Description :**  
Retourne la version de format X.509 d’un certificat.

**Syntaxe :**  
`str CertThumbprint(binary certificateRawData)`  
*   certificateRawData : Représentation en tableau d’octets d’un certificat X.509.Byte array representation of an X.509 certificate. Le tableau d’octets peut contenir des données X.509 codées en binaire (DER) ou en Base64.

---
### <a name="cguid"></a>CGuid
**Description :**  
La fonction CGuid convertit la représentation sous forme de chaîne d’un GUID en sa représentation binaire.

**Syntaxe :**  
`bin CGuid(str GUID)`

* Une chaîne rédigée au format suivant : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx ou {xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}

---
### <a name="contains"></a>Contient
**Description :**  
La fonction Contains détecte une chaîne à l’intérieur d’un attribut à valeurs multiples.

**Syntaxe :**  
`num Contains (mvstring attribute, str search)` - sensible à la casse  
`num Contains (mvstring attribute, str search, enum Casetype)`  
`num Contains (mvref attribute, str search)` - sensible à la casse

* attribut : attribut à valeurs multiples à rechercher.
* recherche : chaîne à rechercher dans l’attribut.
* Casetype : CaseInsensitive ou CaseSensitive.

Renvoie l’indice dans l’attribut à plusieurs valeurs où la chaîne a été trouvée. Si la chaîne est introuvable, la valeur renvoyée est 0.

**Remarques :**  
Pour les attributs de chaîne à valeurs multiples, la recherche détecte des sous-chaînes dans les valeurs.  
Pour les attributs de référence, la chaîne recherchée doit correspondre exactement à la valeur pour être considérée comme une correspondance.

**Exemple :**  
`IIF(Contains([proxyAddresses],"SMTP:")>0,[proxyAddresses],Error("No primary SMTP address found."))`  
Si l’attribut proxyAddresses a une adresse de messagerie principale (indiquée par « SMTP : »), cette fonction renvoie l’attribut proxyAddress. Sinon, elle renvoie une erreur.

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
### <a name="convertfromutf8hex"></a>ConvertFromUTF8Hex
**Description :**  
La fonction ConvertFromUTF8Hex convertit la valeur encodée hexadécimale UTF8 spécifiée en chaîne.

**Syntaxe :**  
`str ConvertFromUTF8Hex(str source)`

* source : Chaîne encodée sur 2 octets UTF8

**Remarques :**  
La différence entre cette fonction et ConvertFromBase64(,UTF8) est que le résultat est convivial pour l’attribut DN.  
Ce format est utilisé par Azure Active Directory en tant que nom de domaine.

**Exemple :**  
`ConvertFromUTF8Hex("48656C6C6F20776F726C6421")`  
Renvoie «*Hello world!* ».

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
### <a name="cnum"></a>CNum
**Description :**  
La fonction CNum prend une chaîne et renvoie un type de données numérique.

**Syntaxe :**  
`num CNum(str value)`

---
### <a name="cref"></a>CRef
**Description :**  
Convertit une chaîne en attribut de référence.

**Syntaxe :**  
`ref CRef(str value)`

**Exemple :**  
`CRef("CN=LC Services,CN=Microsoft,CN=lcspool01,CN=Pools,CN=RTC Service," & %Forest.LDAP%)`

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
### <a name="dateadd"></a>AjDate
**Description :**  
Renvoie un objet Date contenant une date à laquelle un intervalle de temps spécifié a été ajouté.

**Syntaxe :**  
`dt DateAdd(str interval, num value, dt date)`

* intervalle : Expression de chaîne correspondant l’intervalle de temps que vous souhaitez ajouter. La chaîne doit avoir une des valeurs suivantes :
  * yyyy Année
  * q Trimestre
  * m Mois
  * y Jour de l’année
  * s Jour
  * w Jour de la semaine
  * ww Semaine
  * h Heure
  * n Minute
  * s Seconde
* value : Nombre d’unités que vous souhaitez ajouter. Elle peut être positive (pour obtenir des dates dans le futur) ou négative (pour obtenir des dates dans le passé).
* date : DateTime représentant la date à laquelle l’intervalle est ajouté.

**Exemple :**  
`DateAdd("m", 3, CDate("2001-01-01"))`  
Ajoute 3 mois et renvoie une valeur DateTime représentant « 2001-04-01 ».

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
### <a name="dncomponentrev"></a>DNComponentRev
**Description :**  
La fonction DNComponentRev renvoie la valeur d’un composant de nom de domaine spécifié en partant de la droite (fin).

**Syntaxe :**  
`str DNComponentRev(ref dn, num ComponentNumber)`  
`str DNComponentRev(ref dn, num ComponentNumber, enum Options)`

* dn : attribut de référence à interpréter
* ComponentNumber - composant du nom de domaine à retourner
* Options : Contrôleur de domaine – ignorer tous les composants avec « dc = »

**Exemple :**  
Si le nom de domaine est « cn=Joe,ou=Atlanta,ou=GA,ou=US, dc=contoso,dc=com », alors  
`DNComponentRev(CRef([dn]),3)`  
`DNComponentRev(CRef([dn]),1,"DC")`  
Renvoient US.

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
### <a name="escapedncomponent"></a>EscapeDNComponent
**Description :**  
La fonction EscapeDNComponent prend un composant de nom de domaine et l’isole pour qu’il puisse être représenté dans l’annuaire LDAP.

**Syntaxe :**  
`str EscapeDNComponent(str value)`

**Exemple :**  
`EscapeDNComponent("cn=" & [displayName]) & "," & %ForestLDAP%)`  
Permet de s’assurer que l’objet peut être créé dans un annuaire LDAP, même si l’attribut displayName comporte des caractères d’échappement dans LDAP.

---
### <a name="formatdatetime"></a>FormatDateTime
**Description :**  
La fonction FormatDateTime sert à mettre en forme une valeur DateTime en chaîne dans le format spécifié.

**Syntaxe :**  
`str FormatDateTime(dt value, str format)`

* valeur : valeur au format DateTime
* format : chaîne représentant le format à convertir.

**Remarques :**  
Vous trouverez ici les valeurs possibles pour le format : [Formats de date et heure pour la fonction FORMAT](/dax/custom-date-and-time-formats-for-the-format-function).

**Exemple :**  

`FormatDateTime(CDate("12/25/2007"),"yyyy-mm-dd")`  
Renvoie comme résultat « 2007-12-25 ».

`FormatDateTime(DateFromNum([pwdLastSet]),"yyyyMMddHHmmss.0Z")`  
Peut donner comme résultat « 20140905081453.0Z ».

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
### <a name="instrrev"></a>InStrRev
**Description :**  
La fonction InStrRev recherche la dernière occurrence d’une sous-chaîne dans une chaîne.

**Syntaxe :**  
`num InstrRev(str stringcheck, str stringmatch)`  
`num InstrRev(str stringcheck, str stringmatch, num start)`  
`num InstrRev(str stringcheck, str stringmatch, num start, enum compare)`

* stringcheck : chaîne à rechercher
* stringmatch : chaîne à trouver
* start : position de départ pour trouver la sous-chaîne
* compare : vbTextCompare ou vbBinaryCompare

**Remarques :**  
Renvoie la position à laquelle la sous-chaîne a été trouvée, ou 0 si elle est introuvable.

**Exemple :**  
`InStrRev("abbcdbbbef","bb")`  
Renvoie 7.

---
### <a name="isbitset"></a>IsBitSet
**Description :**  
La fonction IsBitSet vérifie si un bit est ou non défini.

**Syntaxe :**  
`bool IsBitSet(num value, num flag)`

* value : valeur numérique évaluée. flag : valeur numérique contenant le bit à évaluer

**Exemple :**  
`IsBitSet(&HF,4)`  
Renvoie True, car le bit « 4 » est défini dans la valeur hexadécimale « F ».

---
### <a name="isdate"></a>IsDate
**Description :**  
La fonction IsDate prend la valeur True si l’expression peut être évaluée à un type DateTime.

**Syntaxe :**  
`bool IsDate(var Expression)`

**Remarques :**  
Permet de déterminer si CDate() peut aboutir.

---
### <a name="iscert"></a>IsCert
**Description :**  
Retourne la valeur true si les données brutes peuvent être sérialisées en un objet certificat .NET X509Certificate2.

**Syntaxe :**  
`bool CertThumbprint(binary certificateRawData)`  
*   certificateRawData : Représentation en tableau d’octets d’un certificat X.509.Byte array representation of an X.509 certificate. Le tableau d’octets peut contenir des données X.509 codées en binaire (DER) ou en Base64.
---
### <a name="isempty"></a>IsEmpty
**Description :**  
La fonction IsEmpty prend la valeur True si l’attribut est présent dans CS ou MV mais qu’il est évalué à une chaîne vide.

**Syntaxe :**  
`bool IsEmpty(var Expression)`

---
### <a name="isguid"></a>IsGuid
**Description :**  
La fonction IsGuid renvoie la valeur True si la chaîne peut être convertie en GUID.

**Syntaxe :**  
`bool IsGuid(str GUID)`

**Remarques :**  
Un GUID est défini en tant que chaîne en fonction de l’un de ces modèles : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx or {xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}.

Utilisé pour déterminer si CGuid() peut aboutir.

**Exemple :**  
`IIF(IsGuid([strAttribute]),CGuid([strAttribute]),NULL)`  
Si StrAttribute est au format GUID, renvoie une représentation binaire. Sinon, renvoie la valeur Null.

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
### <a name="isnumeric"></a>IsNumeric
**Description :**  
La fonction IsNumeric renvoie une valeur booléenne indiquant si une expression peut être évaluée en tant que type de nombre.

**Syntaxe :**  
`bool IsNumeric(var Expression)`

**Remarques :**  
Permet de déterminer si CNum() peut parvenir à analyser l’expression.

---
### <a name="isstring"></a>IsString
**Description :**  
La fonction IsString prend la valeur True si l’expression peut être évaluée en tant que type de chaîne.

**Syntaxe :**  
`bool IsString(var expression)`

**Remarques :**  
Permet de déterminer si CStr() peut parvenir à analyser l’expression.

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
### <a name="itemornull"></a>ItemOrNull
**Description :**  
La fonction ItemOrNull renvoie un élément à partir d’une chaîne/d’un attribut à valeurs multiples.

**Syntaxe :**  
`var ItemOrNull(mvstr attribute, num index)`

* attribute : attribut à valeurs multiples
* index : index vers un élément dans la chaîne à valeurs multiples.

**Remarques :**  
La fonction ItemOrNull est utile avec la fonction Contains, car cette dernière renvoie l’index à un élément de l’attribut à valeurs multiples.

Renvoie une valeur Null si l’index est hors limites.

---
### <a name="join"></a>Join
**Description :**  
La fonction Join prend une chaîne à valeurs multiples et renvoie une chaîne à valeur unique avec le séparateur spécifié inséré entre chaque élément.

**Syntaxe :**  
`str Join(mvstr attribute)`  
`str Join(mvstr attribute, str Delimiter)`

* : Attribut à valeurs multiples contenant des chaînes à joindre.
* delimiter : Toute chaîne utilisée pour séparer les sous-chaînes dans la chaîne renvoyée. En cas d’omission, le caractère espace (" ") est utilisé. Si le délimiteur est une chaîne de longueur nulle ("") ou Nothing, tous les éléments de la liste sont concaténés sans délimiteurs.

**Remarques**  
Il existe une parité entre les fonctions Join et Split. La fonction Join prend un tableau de chaînes et les joint à l’aide d’une chaîne de délimiteur, pour renvoyer une chaîne unique. La fonction Split accepte une chaîne et la sépare au niveau du délimiteur, pour renvoyer un tableau de chaînes. Toutefois, la principale différence est que Join peut concaténer des chaînes avec n’importe quelle chaîne de délimiteur, Split peut uniquement séparer des chaînes à l’aide d’un délimiteur de caractère unique.

**Exemple :**  
`Join([proxyAddresses],",")`  
Peut retourner : « SMTP:john.doe@contoso.com,smtp:jd@contoso.com »

---
### <a name="lcase"></a>Minuscule
**Description :**  
La fonction LCase convertit tous les caractères d’une chaîne en minuscules.

**Syntaxe :**  
`str LCase(str value)`

**Exemple :**  
`LCase("TeSt")`  
Renvoie « test ».

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
Renvoie « Joh ».

---
### <a name="len"></a>NbCar
**Description :**  
La fonction Len renvoie le nombre de caractères contenus dans une chaîne.

**Syntaxe :**  
`num Len(str value)`

**Exemple :**  
`Len("John Doe")`  
Renvoie 8.

---
### <a name="ltrim"></a>SupprGauche
**Description :**  
La fonction LTrim supprime les espaces blancs situés au début d’une chaîne.

**Syntaxe :**  
`str LTrim(str value)`

**Exemple :**  
`LTrim(" Test ")`  
Renvoie « Test ».

---
### <a name="mid"></a>ExtracChaîne
**Description :**  
La fonction Mid renvoie un nombre donné de caractères à partir d’une position spécifiée dans une chaîne.

**Syntaxe :**  
`str Mid(str string, num start, num NumChars)`

* string : chaîne à partir de laquelle les caractères sont renvoyés
* start : nombre identifiant la position de départ dans la chaîne à partir de laquelle les caractères sont renvoyés
* NumChars : nombre identifiant le nombre de caractères à retourner à partir de la position dans la chaîne

**Remarques :**  
Renvoie numChars caractères à partir de la position de départ dans la chaîne.  
Chaîne contenant numChars caractères à partir de la position de départ dans la chaîne :

* Si numChars = 0, retourne une chaîne vide.
* Si numChars < 0, retourne une chaîne d’entrée.
* Si start > la longueur de la chaîne, retourne une chaîne d’entrée.
* Si start < = 0, retourne une chaîne d’entrée.
* Si la chaîne est null, retourne une chaîne vide.

S’il ne reste pas numChars caractères dans la chaîne à partir de la position de départ, autant de caractères que possible sont renvoyés.

**Exemple :**  
`Mid("John Doe", 3, 5)`  
Renvoie « hn Do ».

`Mid("John Doe", 6, 999)`  
Renvoie « Doe ».

---
### <a name="now"></a>maintenant
**Description :**  
La fonction Now renvoie une valeur DateTime indiquant la date et l’heure actuelles qui correspondent à la date et à l’heure système de votre ordinateur.

**Syntaxe :**  
`dt Now()`

---
### <a name="numfromdate"></a>NumFromDate
**Description :**  
La fonction NumFromDate renvoie une date au format de date AD.

**Syntaxe :**  
`num NumFromDate(dt value)`

**Exemple :**  
`NumFromDate(CDate("2012-01-01 23:00:00"))`  
Renvoie 129699324000000000.

---
### <a name="padleft"></a>PadLeft
**Description :**  
La fonction PadLeft remplit par la gauche une chaîne sur une longueur spécifiée à l’aide d’un caractère de remplissage fourni.

**Syntaxe :**  
`str PadLeft(str string, num length, str padCharacter)`

* string : chaîne à remplir.
* length : Entier représentant la longueur de chaîne souhaitée.
* padCharacter : Chaîne constituée d’un seul caractère à utiliser comme caractère de remplissage

**Remarques :**

* Si la longueur de chaîne est inférieure à la longueur length, padCharacter est ajouté à plusieurs reprises au début (à gauche) de la chaîne jusqu’à ce qu’à atteindre la longueur length.
* PadCharacter peut être un caractère d’espacement, mais il ne peut pas s’agir de la valeur null.
* Si la longueur de chaîne est égale ou supérieure à la longueur length, la chaîne est renvoyée inchangée.
* Si la longueur de la chaîne est supérieure ou égale à la longueur length, une chaîne identique est renvoyée.
* Si la longueur de chaîne est inférieure à la longueur length, une nouvelle chaîne de longueur souhaitée est retournée, et contient une chaîne remplie avec un padCharacter.
* Si la chaîne est null, la fonction retourne une chaîne vide.

**Exemple :**  
`PadLeft("User", 10, "0")`  
Renvoie « 000000User ».

---
### <a name="padright"></a>PadRight
**Description :**  
La fonction PadRight remplit par la droite une chaîne sur une longueur spécifiée à l’aide d’un caractère de remplissage fourni.

**Syntaxe :**  
`str PadRight(str string, num length, str padCharacter)`

* string : chaîne à remplir.
* length : Entier représentant la longueur de chaîne souhaitée.
* padCharacter : Chaîne constituée d’un seul caractère à utiliser comme caractère de remplissage

**Remarques :**

* Si la longueur de chaîne est inférieure à la longueur length, padCharacter est ajouté à plusieurs reprises à la fin (à droite) de la chaîne jusqu’à ce qu’à atteindre la longueur length.
* PadCharacter peut être un caractère d’espacement, mais il ne peut pas s’agir de la valeur null.
* Si la longueur de chaîne est égale ou supérieure à la longueur length, la chaîne est renvoyée inchangée.
* Si la longueur de la chaîne est supérieure ou égale à la longueur length, une chaîne identique est renvoyée.
* Si la longueur de chaîne est inférieure à la longueur length, une nouvelle chaîne de longueur souhaitée est retournée, et contient une chaîne remplie avec un padCharacter.
* Si la chaîne est null, la fonction retourne une chaîne vide.

**Exemple :**  
`PadRight("User", 10, "0")`  
Renvoie « User000000 ».

---
### <a name="pcase"></a>PCase
**Description :**  
La fonction PCase met en majuscule le premier caractère de chaque mot délimité par un espace dans une chaîne, et tous les autres caractères sont convertis en minuscules.

**Syntaxe :**  
`String PCase(string)`

**Remarques :**

* Cette fonction ne fournit pas pour le moment de casse appropriée pour convertir un mot qui est entièrement en majuscules, par exemple un sigle.

**Exemple :**  
`PCase("TEsT")`  
Renvoie « test ».

`PCase(LCase("TEST"))`  
Renvoie « Test ».

---
### <a name="randomnum"></a>RandomNum
**Description :**  
La fonction RandomNum renvoie un nombre aléatoire dans un intervalle spécifié.

**Syntaxe :**  
`num RandomNum(num start, num end)`

* start : nombre identifiant la limite inférieure de la valeur aléatoire à générer
* end : nombre identifiant la limite supérieure de la valeur aléatoire à générer

**Exemple :**  
`Random(100,999)`  
Peut renvoyer 734.

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
**Description :**  
La fonction Replace remplace toutes les occurrences d’une chaîne par une autre chaîne.

**Syntaxe :**  
`str Replace(str string, str OldValue, str NewValue)`

* string : Chaîne dans laquelle les caractères sont remplacés.
* OldValue : Chaîne à rechercher et remplacer.
* NewValue : Chaîne de remplacement.

**Remarques :**  
La fonction reconnaît les monikers spéciaux suivants :

* \n – Nouvelle ligne
* \r – Retour chariot
* \t – Tabulation

**Exemple :**  
`Replace([address],"\r\n",", ")`  
Remplace CRLF par une virgule et un espace, et peut générer « One Microsoft Way, Redmond, WA, USA ».

---
### <a name="replacechars"></a>ReplaceChars
**Description :**  
La fonction ReplaceChars remplace toutes les occurrences des caractères trouvés dans la chaîne ReplacePattern.

**Syntaxe :**  
`str ReplaceChars(str string, str ReplacePattern)`

* string : Chaîne dans laquelle les caractères sont remplacés.
* ReplacePattern : chaîne contenant un dictionnaire avec des caractères à remplacer.

Le format est {source1}: {target1}, {source2}: {target2}, {sourceN}, {targetN}, source étant le caractère à rechercher et la cible, la chaîne à remplacer.

**Remarques :**

* La fonction prend chaque occurrence de sources définies et la remplace par les cibles.
* La source doit être exactement un caractère (unicode).
* La source ne peut pas être vide ou dépasser un caractère (erreur d’analyse).
* La cible peut contenir plusieurs caractères, par exemple ö:oe, β:ss.
* La cible peut être vide, indiquant que le caractère doit être supprimé.
* La source respecte la casse et il doit s’agir d’une correspondance exacte.
* La , (Virgule) et : (deux-points) sont des caractères réservés et ne peuvent pas être remplacés avec cette fonction.
* Les espaces et autres caractères blancs dans la chaîne ReplacePattern sont ignorés.

**Exemple :**  
`%ReplaceString% = ’:,Å:A,Ä:A,Ö:O,å:a,ä:a,ö,o`

`ReplaceChars("Räksmörgås",%ReplaceString%)`  
Renvoie Raksmorgas.

`ReplaceChars("O’Neil",%ReplaceString%)`  
Renvoie « ONeil », l’apostrophe est définie comme étant à supprimer.

---
### <a name="right"></a>Right
**Description :**  
La fonction Right renvoie un nombre spécifié de caractères en partant de la droite (fin) d’une chaîne.

**Syntaxe :**  
`str Right(str string, num NumChars)`

* string : chaîne à partir de laquelle les caractères sont renvoyés
* numChars : nombre identifiant le nombre de caractères à retourner à partir de la fin (à droite) de la chaîne

**Remarques :**  
Les numChars caractères sont renvoyés à partir de la dernière position de la chaîne.

Chaîne contenant les numChars derniers caractères de la chaîne :

* Si numChars = 0, retourne une chaîne vide.
* Si numChars < 0, retourne une chaîne d’entrée.
* Si la chaîne est null, retourne une chaîne vide.

Si la chaîne contient un nombre de caractères inférieur au nombre spécifié dans numChars, une chaîne identique est renvoyée.

**Exemple :**  
`Right("John Doe", 3)`  
Renvoie « Doe ».

---
### <a name="rtrim"></a>SupprDroite
**Description :**  
La fonction RTrim supprime les espaces blancs situés à la fin d’une chaîne.

**Syntaxe :**  
`str RTrim(str value)`

**Exemple :**  
`RTrim(" Test ")`  
Renvoie « Test ».

---
### <a name="select"></a>Sélectionnez
**Description :**  
Traite toutes les valeurs d’un attribut à valeurs multiples (ou la sortie d’une expression) selon la fonction spécifiée.

**Syntaxe :**  
`mvattr Select(variable item, mvattr attribute, func function)`  
`mvattr Select(variable item, exp expression, func function)`

* item : Représente un élément de l’attribut à valeurs multiples
* attribute : l’attribut à valeurs multiples
* expression : expression qui retourne une collection de valeurs
* condition : toute fonction pouvant traiter un élément de l’attribut

**Exemples :**  
`Select($item,[otherPhone],Replace($item,"-",""))`  
Retourne toutes les valeurs de l’attribut à valeurs multiples otherPhone après suppression des traits d’union (-).

---
### <a name="split"></a>Split
**Description :**  
La fonction Split prend une chaîne séparée par un délimiteur et en fait une chaîne à valeurs multiples.

**Syntaxe :**  
`mvstr Split(str value, str delimiter)`  
`mvstr Split(str value, str delimiter, num limit)`

* value : chaîne contenant un caractère délimiteur pour assurer la séparation.
* delimiter : caractère unique à utiliser comme délimiteur.
* limit : nombre maximal de valeurs qu’il est possible de renvoyer.

**Exemple :**  
`Split("SMTP:john.doe@contoso.com,smtp:jd@contoso.com",",")`  
Renvoie une chaîne à valeurs multiples avec 2 éléments utiles pour l’attribut proxyAddress.

---
### <a name="stringfromguid"></a>StringFromGuid
**Description :**  
La fonction StringFromGuid prend un GUID binaire et le convertit en chaîne.

**Syntaxe :**  
`str StringFromGuid(bin GUID)`

---
### <a name="stringfromsid"></a>StringFromSid
**Description :**  
La fonction StringFromSid convertit en chaîne un tableau d’octets contenant un identificateur de sécurité.

**Syntaxe :**  
`str StringFromSid(bin ObjectSID)`  

---
### <a name="switch"></a>Commutateur
**Description :**  
La fonction Switch est utilisée pour renvoyer une valeur unique en fonction des conditions évaluées.

**Syntaxe :**  
`var Switch(exp expr1, var value1[, exp expr2, var value … [, exp expr, var valueN]])`

* expr : Expression de variante à évaluer.
* value : Valeur à retourner si l’expression correspondante a la valeur True.

**Remarques :**  
La liste d’arguments de la fonction Switch se compose de paires d’expressions et de valeurs. Les expressions sont évaluées de gauche à droite et la valeur associée à la première expression à évoluer à True est renvoyée. Si les parties ne sont pas correctement couplées, une erreur d’exécution se produit.

Par exemple, si expr1 est True, Switch renvoie la valeur1. Si expr-1 est False, mais expr-2 est True, Switch renvoie la valeur 2, et ainsi de suite.

Switch ne renvoie rien si :

* Aucune des expressions n’est vraie.
* La première expression True possède une valeur correspondante Null.

Switch évalue toutes les expressions, même si elle n’en renvoie qu’une. Pour cette raison, il est conseillé de surveiller les éventuels effets secondaires. Par exemple, si l’évaluation d’une expression entraîne une division par zéro, une erreur se produit.

La valeur peut être également la fonction Error qui renvoie une chaîne personnalisée.

**Exemple :**  
`Switch([city] = "London", "English", [city] = "Rome", "Italian", [city] = "Paris", "French", True, Error("Unknown city"))`  
Renvoie la langue parlée dans certaines grandes villes ; sinon, renvoie une erreur.

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
### <a name="ucase"></a>UCase
**Description :**  
La fonction UCase convertit tous les caractères d’une chaîne en majuscules.

**Syntaxe :**  
`str UCase(str string)`

**Exemple :**  
`UCase("TeSt")`  
Renvoie « test ».

---
### <a name="where"></a>Where

**Description :**  
Retourne un sous-ensemble de valeurs d’un attribut à valeurs multiples (ou la sortie d’une expression) en fonction de la condition.

**Syntaxe :**  
`mvattr Where(variable item, mvattr attribute, exp condition)`  
`mvattr Where(variable item, exp expression, exp condition)`  
* item : Représente un élément de l’attribut à valeurs multiples
* attribute : l’attribut à valeurs multiples
* condition : toute expression pouvant avoir la valeur true ou false
* expression : expression qui retourne une collection de valeurs

**Exemple :**  
`Where($item,[userCertificate],CertNotAfter($item)>Now())`  
Retourne les valeurs de certificat de l’attribut à valeurs multiples userCertificate qui n’ont pas expiré.

---
### <a name="with"></a>With
**Description :**  
La fonction With permet de simplifier une expression complexe en utilisant une variable pour représenter une sous-expression qui apparaît une ou plusieurs fois dans l’expression complexe.

**Syntaxe**
`With(var variable, exp subExpression, exp complexExpression)` :  
* variable : Représente la sous-expression.
* subExpression : sous-expression représentée par une variable.
* complexExpression : Expression complexe.

**Exemple :**  
`With($unExpiredCerts,Where($item,[userCertificate],CertNotAfter($item)>Now()),IIF(Count($unExpiredCerts)>0,$unExpiredCerts,NULL))`  
Fonctionnellement équivalent à :  
`IIF (Count(Where($item,[userCertificate],CertNotAfter($item)>Now()))>0, Where($item,[userCertificate],CertNotAfter($item)>Now()),NULL)`  
Qui retourne uniquement les valeurs de certificat non expirées de l’attribut userCertificate.


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

## <a name="additional-resources"></a>Ressources supplémentaires
* [Comprendre les expressions d’approvisionnement déclaratif](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md)
* [Synchronisation Azure AD Connect : personnaliser les options de synchronisation](how-to-connect-sync-whatis.md)
* [Intégration des identités locales dans Azure Active Directory](whatis-hybrid-identity.md)