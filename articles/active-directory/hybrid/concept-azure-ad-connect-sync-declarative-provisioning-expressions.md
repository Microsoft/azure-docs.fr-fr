---
title: 'Azure AD Connect : Expressions d’approvisionnement déclaratif | Microsoft Docs'
description: Explique les expressions d’approvisionnement déclaratif.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: e3ea53c8-3801-4acf-a297-0fb9bb1bf11d
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/18/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 02490839a9e35695ae2e8b3f750e139ad7413aa4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89280211"
---
# <a name="azure-ad-connect-sync-understanding-declarative-provisioning-expressions"></a>Azure AD Connect Sync : présentation des expressions d’approvisionnement déclaratif
Azure AD Connect sync s’appuie sur l’approvisionnement déclaratif introduit pour la première fois dans Forefront Identity Manager 2010. Il vous permet d’implémenter toute votre logique métier d’intégration des identités sans avoir à écrire de code compilé.

Le langage d’expression utilisé dans les flux d’attributs constitue une partie essentielle de l’approvisionnement déclaratif. Le langage utilisé est un sous-ensemble de Microsoft® Visual Basic® pour Applications (VBA). Ce langage est utilisé dans Microsoft Office et les utilisateurs qui ont une certaine expérience de VBScript le reconnaîtront également. Le langage d’expression de l’approvisionnement déclaratif utilise uniquement des fonctions. Il ne s’agit pas d’un langage structuré. Il ne contient ni méthodes ni instructions. Les fonctions sont imbriquées pour exprimer le déroulement du programme.

Pour plus d’informations, consultez [Bienvenue dans la référence linguistique Visual Basic pour Applications pour Office 2013](/office/vba/api/overview/language-reference).

Les attributs sont fortement typés. Une fonction accepte uniquement les attributs de type correct. Elle respecte également la casse. Les noms des fonctions et les noms des attributs doivent avoir une casse appropriée, sinon une erreur est levée.

## <a name="language-definitions-and-identifiers"></a>Définitions linguistiques et identificateurs
* Les fonctions ont un nom suivi d’arguments entre parenthèses : NomFonction (argument 1, argument N).
* Les attributs sont identifiés par des crochets : [NomAttribut]
* Les paramètres sont identifiés par des signes de pourcentage : %NomParamètre%
* Les constantes de chaîne sont placées entre guillemets : par exemple, "Contoso" (Remarque : vous devez utiliser des guillemets droits "" et non pas des guillemets courbes “”)
* Les valeurs numériques sont exprimées sans guillemets et les valeurs attendues sont décimales. Les valeurs hexadécimales sont précédées de &H. Par exemple, 98052, &HFF
* Les valeurs booléennes sont exprimées avec des constantes : True, False.
* Les constantes intégrées et les littéraux sont exprimés uniquement par leur nom : NULL, CRLF, IgnoreThisFlow

### <a name="functions"></a>Fonctions
L’approvisionnement déclaratif utilise de nombreuses fonctions pour permettre de transformer les valeurs d’attribut. Elles peuvent être imbriquées. Le résultat d’une fonction est alors transmis à une autre fonction.

`Function1(Function2(Function3()))`

Vous trouverez la liste complète des fonctions dans la [référence de fonction](reference-connect-sync-functions-reference.md).

### <a name="parameters"></a>Paramètres
Un paramètre est défini par un connecteur ou par un administrateur à l’aide de PowerShell. Les paramètres contiennent généralement des valeurs qui diffèrent d’un système à un autre, par exemple le nom du domaine où se trouve l’utilisateur. Vous pouvez les utiliser dans des flux d’attributs.

Le connecteur Active Directory fournissait les paramètres suivants pour les règles de synchronisation entrantes :

| Nom du paramètre | Commentaire |
| --- | --- |
| Domain.Netbios |Format NetBIOS du domaine en cours d'importation, par exemple, FABRIKAMSALES |
| Domain.FQDN |Format FQDN du domaine en cours d’importation, par exemple, sales.fabrikam.com |
| Domain.LDAP |Format LDAP du domaine en cours d’importation, par exemple, DC=sales,DC=fabrikam,DC=com |
| Forest.Netbios |Format NetBIOS du nom de forêt en cours d’importation, par exemple, FABRIKAMCORP |
| Forest.FQDN |Format FQDN du nom de forêt en cours d’importation, par exemple, fabrikam.com |
| Forest.LDAP |Format LDAP du nom de forêt en cours d’importation, par exemple, DC=fabrikam,DC=com |

Le système fournit le paramètre suivant, qui est utilisé pour obtenir l'identificateur du connecteur en cours d'exécution :  
`Connector.ID`

Voici un exemple qui remplira le domaine d’attribut du métaverse avec le nom netbios du domaine où se trouve l’utilisateur :  
`domain` <- `%Domain.Netbios%`

### <a name="operators"></a>Opérateurs
Vous pouvez utiliser les opérateurs suivants :

* **Opérateurs de comparaison** : &lt;, &lt;=, &lt;&gt;, =, &gt;, &gt;=
* **Mathématiques** : +, -, \*, -
* **Chaîne** : &amp; (concaténation)
* **Logiques** : &amp;&amp; (et), || (ou)
* **Ordre d’évaluation**: ( )

Les opérateurs sont évalués de la gauche vers la droite et ont la même priorité d'évaluation. Par exemple, le \* (signe multiplicateur) n’est pas évalué avant - (soustraction). 2\*(5+3) n’est pas la même chose que 2\*5+3. Les parenthèses ( ) sont utilisées pour modifier l’ordre d’évaluation lorsqu’un ordre d’évaluation de la gauche vers la droite n’est pas approprié.

## <a name="multi-valued-attributes"></a>Attributs à valeurs multiples
Les fonctions peuvent fonctionner sur des attributs à valeur unique et à valeurs multiples. Pour les attributs à valeurs multiples, la fonction fonctionne sur toutes les valeurs et applique la même fonction à chaque valeur.

Par exemple :  
`Trim([proxyAddresses])` supprime les espaces de gauche à droite dans chaque valeur de l’attribut proxyAddress.  
`Word([proxyAddresses],1,"@") & "@contoso.com"` Pour chaque valeur comportant un signe @-sign, remplacez le domaine par @contoso.com.  
`IIF(InStr([proxyAddresses],"SIP:")=1,NULL,[proxyAddresses])` Recherchez l’adresse SIP et supprimez-la des valeurs.

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur le modèle de configuration dans [Comprendre l’approvisionnement déclaratif](concept-azure-ad-connect-sync-declarative-provisioning.md).
* Apprendre comment l’approvisionnement déclaratif est utilisé out-of-box dans [Présentation de la configuration par défaut](concept-azure-ad-connect-sync-default-configuration.md).
* Apprendre à effectuer une modification pratique à l’aide de l’approvisionnement déclaratif dans [Comment modifier la configuration par défaut](how-to-connect-sync-change-the-configuration.md).

**Rubriques de présentation**

* [Azure AD Connect Sync - Présentation et personnalisation des options de synchronisation](how-to-connect-sync-whatis.md)
* [Intégration des identités locales dans Azure Active Directory](whatis-hybrid-identity.md)

**Rubriques de référence**

* [Azure AD Connect Sync : Référence aux fonctions](reference-connect-sync-functions-reference.md)