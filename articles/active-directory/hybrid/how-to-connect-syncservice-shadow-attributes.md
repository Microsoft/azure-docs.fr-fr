---
title: Attributs des clichés instantanés du service de synchronisation AD Connect | Microsoft Docs
description: Décrit le fonctionnement des attributs des clichés instantanés dans le service de synchronisation Azure AD Connect.
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
ms.topic: how-to
ms.date: 07/13/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 128303cb51b39db8442fdda71f949db17923bfa2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90088968"
---
# <a name="azure-ad-connect-sync-service-shadow-attributes"></a>Attributs de clichés instantanés du service de synchronisation Azure AD Connect
La plupart des attributs sont représentés de la même façon dans Azure AD qu’ils le sont dans votre Active Directory local. Toutefois, certains attributs ont une gestion spéciale, et la valeur d’attribut dans Azure AD peut être différente de ce qu’Azure AD Connect synchronise.

## <a name="introducing-shadow-attributes"></a>Présentation des attributs des clichés instantanés
Certains attributs ont deux représentations dans Azure AD. La valeur locale et une valeur calculée sont stockées. Ces attributs supplémentaires sont appelés attributs de clichés instantanés. Les deux attributs les plus courants pour lesquels vous voyez ce comportement sont **userPrincipalName** et **proxyAddress**. La modification des valeurs d’attribut se produit lorsque des valeurs de ces attributs représentent des domaines non vérifiés. Mais le moteur de synchronisation de Connect lit la valeur de l’attribut de cliché instantané. Ainsi, de son point de vue, l’attribut a été confirmé par Azure AD.

Vous ne voyez pas les attributs de cliché instantané en utilisant le portail Azure ou avec PowerShell. Mais comprendre le concept vous aide à corriger certains scénarios où l’attribut a des valeurs différentes en local et dans le cloud.

Pour mieux comprendre le comportement, observez cet exemple de Fabrikam :  
![Capture d'écran montrant le suffixe UPN Active Directory de plusieurs exemples, avec la valeur de domaine Azure AD correspondante Non ajouté, Non vérifié ou Vérifié.](./media/how-to-connect-syncservice-shadow-attributes/domains.png)  
Ils ont plusieurs suffixes UPN dans leur Active Directory local, mais ils n’en ont vérifié qu’un.

### <a name="userprincipalname"></a>userPrincipalName
Un utilisateur possède les valeurs d’attribut suivantes dans un domaine non vérifié :

| Attribut | Valeur |
| --- | --- |
| userPrincipalName local | lee.sperry@fabrikam.com |
| shadowUserPrincipalName dans Azure AD | lee.sperry@fabrikam.com |
| userPrincipalName dans Azure AD | lee.sperry@fabrikam.onmicrosoft.com |

L’attribut userPrincipalName est la valeur que vous voyez lors de l’utilisation de PowerShell.

Étant donné que la valeur d’attribut locale réelle est stockée dans Azure AD, lorsque vous vérifiez le domaine fabrikam.com, Azure AD met à jour l’attribut userPrincipalName avec la valeur de shadowUserPrincipalName. Vous n’avez pas à synchroniser les modifications apportées à Azure AD Connect pour que ces valeurs soient mises à jour.

### <a name="proxyaddresses"></a>proxyAddresses
Le même processus pour inclure uniquement les domaines vérifiés se produit également pour proxyAddresses, mais avec une logique supplémentaire. La confirmation des domaines vérifiés se produit uniquement pour les utilisateurs de boîtes aux lettres. Un utilisateur ou un contact utilisant une boîte aux lettres représente un utilisateur dans une autre organisation Exchange, et vous pouvez ajouter toute valeur présente dans proxyAddresses à ces objets.

Pour un utilisateur de boîte aux lettres, en local ou dans Exchange Online, seules les valeurs pour les domaines vérifiés s’affichent. Vous devriez voir quelque chose de semblable à ceci :

| Attribut | Valeur |
| --- | --- |
| proxyAddresses local | SMTP:abbie.spencer@fabrikamonline.com</br>smtp:abbie.spencer@fabrikam.com</br>smtp:abbie@fabrikamonline.com |
| proxyAddresses dans Exchange Online | SMTP:abbie.spencer@fabrikamonline.com</br>smtp:abbie@fabrikamonline.com</br>SIP:abbie.spencer@fabrikamonline.com |

Dans ce cas **smtp:abbie.spencer\@fabrikam.com** a été supprimé, car ce domaine n’a pas été vérifié. Mais Exchange a également ajouté **SIP:abbie.spencer\@fabrikamonline.com**. Fabrikam n’a pas utilisé Lync/Skype en local, mais Azure AD et Exchange Online s’y préparent.

Cette logique pour proxyAddresses est appelée **ProxyCalc**. ProxyCalc est appelé à chaque modification d’un utilisateur lorsque :

- L’utilisateur s’est vu attribuer à un plan de service qui inclut Exchange Online, même s’il n’avait pas de licence pour Exchange. Par exemple, si l’utilisateur est affecté à la référence (SKU) Office E3, mais s’est seulement vu attribuer SharePoint Online. Cela est vrai même si votre boîte aux lettres est toujours locale.
- L’attribut msExchRecipientTypeDetails a une valeur.
- Vous apportez une modification à proxyAddresses ou userPrincipalName.

ProxyCalc peut avoir besoin d’un certain temps pour traiter une modification sur un utilisateur et n’est pas synchrone avec le processus d’exportation d’Azure AD Connect.

> [!NOTE]
> La logique de ProxyCalc a des comportements supplémentaires pour des scénarios avancés qui ne sont pas documentés dans cette rubrique. Cette rubrique est fournie pour vous aider à comprendre le comportement et non pour détailler l’ensemble de la logique interne.

### <a name="quarantined-attribute-values"></a>Valeurs d’attribut en quarantaine
Les attributs de clichés instantanés sont également utilisés lorsque des valeurs d’attribut sont en double. Pour plus d’informations, consultez [Résilience d’attribut en double](how-to-connect-syncservice-duplicate-attribute-resiliency.md).

## <a name="see-also"></a>Voir aussi
* [Synchronisation d’Azure AD Connect](how-to-connect-sync-whatis.md)
* [Intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).
