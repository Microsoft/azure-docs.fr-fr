---
title: Azure AD Connect Health - Diagnostiquer des erreurs de synchronisation d’attribut en double | Microsoft Docs
description: Ce document décrit le processus de diagnostic des erreurs de synchronisation d’attribut en double et un correctif potentiel des scénarios d’objets orphelins directement à partir du portail Azure.
services: active-directory
documentationcenter: ''
author: zhiweiwangmsft
manager: maheshu
editor: billmath
ms.service: active-directory
ms.subservice: hybrid
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/11/2018
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: c2bd2e72b05cc01b1a351880d565323662635364
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89278681"
---
# <a name="diagnose-and-remediate-duplicated-attribute-sync-errors"></a>Diagnostiquer et corriger les erreurs de synchronisation d’attribut en double

## <a name="overview"></a>Vue d’ensemble
Allant plus loin dans la mise en évidence des erreurs de synchronisation, Azure Active Directory (Azure AD) Connect Health permet la correction en libre-service des erreurs. Il résout les erreurs de synchronisation d’attribut en double et corrige les objets qui sont orphelins à partir d’Azure AD.
La fonctionnalité de diagnostic présente les avantages suivants :
- Elle fournit une procédure de diagnostic qui limite les erreurs de synchronisation d’attribut en double. Elle fournit également des correctifs spécifiques.
- Elle applique un correctif pour des scénarios dédiés à partir d’Azure AD pour résoudre l’erreur en une seule étape.
- Aucune mise à niveau ou configuration n’est nécessaire pour activer cette fonctionnalité.
Pour plus d’informations sur Azure AD, consultez [Synchronisation des identités et résilience d’attribut en double](how-to-connect-syncservice-duplicate-attribute-resiliency.md).

## <a name="problems"></a>Problèmes
### <a name="a-common-scenario"></a>Un scénario courant
Quand des erreurs de synchronisation de **QuarantinedAttributeValueMustBeUnique** et de **AttributeValueMustBeUnique** se produisent, il est courant de voir un conflit de**UserPrincipalName** ou de **Proxy Addresses** dans Azure AD. Vous pouvez résoudre les erreurs de synchronisation en mettant à jour l’objet source en conflit à partir du côté local. L’erreur de synchronisation sera résolue après la synchronisation suivante. Par exemple, cette image indique que deux utilisateurs rencontrent un conflit de leur nom d’utilisateur principal (**UserPrincipalName**). Tous deux possèdent le nom **Joe.J\@contoso.com**. Les objets en conflit sont mis en quarantaine dans Azure AD.

![Scénario courant de diagnostic d’erreur de synchronisation](./media/how-to-connect-health-diagnose-sync-errors/IIdFixCommonCase.png)

### <a name="orphaned-object-scenario"></a>Scénario d’objet orphelin
Parfois, vous pouvez trouver qu’un utilisateur existant perd **l’ancre source**. La suppression de l’objet source s’est produite dans Active Directory local. Mais le changement de signal de suppression n’a jamais été synchronisé avec Azure AD. Cette perte se produit pour des raisons comme des problèmes de moteur de synchronisation ou une migration de domaine. Quand le même objet est restauré ou recréé, un utilisateur existant doit logiquement être l’utilisateur à synchroniser à partir de **l’ancre source**. 

Quand un utilisateur existant est un objet cloud uniquement, vous pouvez également voir l’utilisateur en conflit synchronisé avec Azure AD. L’utilisateur ne peut pas être mis en correspondance avec l’objet existant lors de la synchronisation. Il n’existe aucun moyen direct pour remapper **l’ancre source**. En savoir plus sur la [base de connaissances existante](https://support.microsoft.com/help/2647098). 

À titre d’exemple, l’objet existant dans Azure AD conserve la licence de Joes. Un objet qui vient d’être synchronisé avec une **ancre source** différente se présente dans un état d’attribut en double dans Azure AD. Les changements apportés à Joe sur Active Directory local ne sont pas appliqués à l’utilisateur d’origine de Joe (objet existant) dans Azure AD.  

![Scénario Diagnostiquer une erreur de synchronisation - objet orphelin](./media/how-to-connect-health-diagnose-sync-errors/IIdFixOrphanedCase.png)

## <a name="diagnostic-and-troubleshooting-steps-in-connect-health"></a>Étapes de diagnostic et de dépannage dans Connect Health 
La fonctionnalité de diagnostic prend en charge les objets utilisateur avec les attributs dupliqués suivants :

| Nom de l’attribut | Types d’erreur de synchronisation|
| ------------------ | -----------------|
| UserPrincipalName | QuarantinedAttributeValueMustBeUnique ou AttributeValueMustBeUnique | 
| ProxyAddresses | QuarantinedAttributeValueMustBeUnique ou AttributeValueMustBeUnique | 
| SipProxyAddress | AttributeValueMustBeUnique | 
| OnPremiseSecurityIdentifier |  AttributeValueMustBeUnique |

>[!IMPORTANT]
> Pour accéder à cette fonctionnalité, l’autorisation **Administrateur général** ou **Contributeur** d’Azure RBAC est nécessaire.
>

Suivez les étapes à partir du portail Azure pour affiner les détails des erreurs de synchronisation et fournir des solutions plus spécifiques :

![Étapes de diagnostic d’erreur de synchronisation](./media/how-to-connect-health-diagnose-sync-errors/IIdFixSteps.png)

À partir du portail Azure, effectuez quelques étapes pour identifier des scénarios de réparation spécifiques :  
1.  Vérifiez la colonne **État de diagnostic**. L’état indique s’il existe un moyen de corriger une erreur de synchronisation directement à partir d’Azure Active Directory. En d’autres termes, il existe un flux de résolution des problèmes pour cibler l’erreur et essayer de la corriger.

| Statut | Qu’est-ce que cela signifie ? |
| ------------------ | -----------------|
| Non commencé | Vous n’avez pas visité ce processus de diagnostic. Selon le résultat du diagnostic, il existe un éventuel moyen de résoudre l’erreur de synchronisation directement à partir du portail. |
| Correction manuelle requise | L’erreur ne correspond pas aux critères des correctifs disponibles dans le portail. Soit les types d’objets en conflit ne sont pas des utilisateurs, soit vous avez déjà parcouru les étapes de diagnostic et aucune résolution n’était disponible à partir du portail. Dans ce dernier cas, un correctif à partir du côté local est toujours l’une des solutions. [En savoir plus sur les correctifs locaux](https://support.microsoft.com/help/2647098). | 
| En attente de synchronisation | Un correctif a été appliqué. Le portail attend le prochain cycle de synchronisation pour effacer l’erreur. |

  >[!IMPORTANT]
  > La colonne d’état de diagnostic est réinitialisée après chaque cycle de synchronisation. 
  >

1. Sélectionnez le bouton **Diagnostiquer** situé sous les détails de l’erreur. Vous allez répondre à quelques questions et identifier les détails de l’erreur de synchronisation. Les réponses aux questions permettent d’identifier un cas d’objet orphelin.

1. Si un bouton **Fermer** s’affiche à la fin des diagnostics, aucun correctif rapide n’est disponible à partir du portail, d’après les réponses que vous avez données. Reportez-vous à la solution décrite dans la dernière étape. Les correctifs disponibles en local sont toujours les solutions. Sélectionnez le bouton **Fermer**. L’état de l’erreur de synchronisation actuelle passe à **Correction manuelle requise**. L’état reste inchangé pendant le cycle de synchronisation en cours.

1. Une fois qu’un cas d’objet orphelin est identifié, vous pouvez résoudre les erreurs de synchronisation d’attribut en double directement à partir du portail. Pour déclencher le processus, sélectionnez le bouton **Appliquer le correctif**. L’état de l’erreur de synchronisation en cours est mis à jour pour devenir **En attente de synchronisation**.

1. Après le cycle de synchronisation suivant, l’erreur doit être supprimée de la liste.

## <a name="how-to-answer-the-diagnosis-questions"></a>Comment répondre aux questions de diagnostic 
### <a name="does-the-user-exist-in-your-on-premises-active-directory"></a>Est-ce que l’utilisateur existe dans votre Active Directory local ?

Cette question tente d’identifier l’objet source de l’utilisateur existant à partir d’Active Directory local.  
1. Vérifiez si Azure Active Directory contient un objet ayant le nom d’utilisateur principal (**UserPrincipalName**) indiqué. Si ce n’est pas le cas, répondez **Non**.
2. Si c’est le cas, vérifiez si l’objet est encore concerné par la synchronisation.  
   - Recherchez dans l’espace connecteur Azure AD à l’aide du nom de domaine.
   - Si l’objet est trouvé avec l’état **Ajout en attente**, répondez **Non**. Azure AD Connect ne peut pas connecter l’objet à l’objet AD approprié.
   - Si l’objet est introuvable, répondez **Oui**.

Dans ces exemples, la question tente d’identifier si **Joe Jackson** existe toujours dans Active Directory local.
Pour le **scénario courant**, les deux utilisateurs **Joe Johnson** et **Joe Jackson** sont présents dans Active Directory local. Les objets mis en quarantaine sont deux utilisateurs différents.

![Scénario courant de diagnostic d’erreur de synchronisation](./media/how-to-connect-health-diagnose-sync-errors/IIdFixCommonCase.png)

Pour le **scénario d’objet orphelin**, seul un utilisateur unique **Joe Johnson** est présent dans Active Directory local :

![Scénario Diagnostiquer une erreur de synchronisation - objet orphelin *l’utilisateur existe-t-il*](./media/how-to-connect-health-diagnose-sync-errors/IIdFixOrphanedCase.png)

### <a name="do-both-of-these-accounts-belong-to-the-same-user"></a>Ces deux comptes appartiennent-ils au même utilisateur ?
Cette question vérifie si un utilisateur entrant en conflit et l’objet utilisateur existant dans Azure AD appartiennent au même utilisateur.  
1. L’objet en conflit vient d’être synchronisé avec Azure Active Directory. Comparez les attributs des objets :  
   - Nom d’affichage
   - Nom d’utilisateur principal
   - ID de l'objet
2. Si Azure AD ne parvient pas à les comparer, vérifiez si Active Directory contient des objets ayant les valeurs **UserPrincipalNames** indiquées. Répondez **Non** si vous trouvez les deux.

Dans l’exemple suivant, les deux objets appartiennent au même utilisateur (**Joe Johnson**).

![Scénario Diagnostiquer une erreur de synchronisation - objet orphelin *même utilisateur*](./media/how-to-connect-health-diagnose-sync-errors/IIdFixOrphanedCase.png)


## <a name="what-happens-after-the-fix-is-applied-in-the-orphaned-object-scenario"></a>Que se passe-t-il une fois le correctif appliqué dans le scénario d’objet orphelin ?
En fonction des réponses aux questions précédentes, le bouton **Appliquer un correctif** s’affiche quand un correctif est disponible à partir d’Azure AD. Dans ce cas, l’objet local est synchronisé avec un objet Azure AD inattendu. Les deux objets sont mappés à l’aide de **l’ancre source**. Le changement **Appliquer le correctif** suit les étapes suivantes ou des étapes similaires :
1. Met à jour **l’ancre source** sur l’objet approprié dans Azure AD.
2. Supprime l’objet en conflit dans Azure AD, s’il est présent.

![Diagnostiquer une erreur de synchronisation après la correction](./media/how-to-connect-health-diagnose-sync-errors/IIdFixAfterFix.png)

>[!IMPORTANT]
> Le changement **Appliquer le correctif** s’applique uniquement aux cas d’objet orphelin.
>

Une fois les étapes précédentes effectuées, l’utilisateur peut accéder à la ressource d’origine, qui est un lien vers un objet existant. La valeur **État de diagnostic** dans la liste est mise à jour pour devenir **En attente de synchronisation**. L’erreur de synchronisation sera résolue après la synchronisation suivante. Connect Health n’affiche plus l’erreur de synchronisation résolue dans la liste.

## <a name="failures-and-error-messages"></a>Échecs et messages d’erreur
**Un utilisateur avec un attribut en conflit est supprimé temporairement d’Azure Active Directory. Vérifiez que l’utilisateur est définitivement supprimé avant de réessayer.**  
Un utilisateur avec un attribut en conflit dans Azure AD doit être effacé avant d’appliquer le correctif. Découvrez [comment supprimer définitivement un utilisateur dans Azure AD](../fundamentals/active-directory-users-restore.md) avant de réappliquer le correctif. Par ailleurs, l’utilisateur est automatiquement supprimé définitivement 30 jours après avoir été supprimé temporairement. 

**La mise à jour de l'ancre source vers un utilisateur basé sur le cloud n'est pas prise en charge dans votre client.**  
Un utilisateur basé sur le cloud dans Azure AD ne doit pas avoir d’ancre source. Dans ce cas, la mise à jour de l’ancre source n’est pas prise en charge. Une correction manuelle doit être effectuée localement. 

## <a name="faq"></a>Questions fréquentes (FAQ)
**Q.** Que se passe-t-il en cas d’échec de l’**application du correctif** ?  
**A.** Si l’exécution échoue, il est possible qu’Azure AD Connect fasse l’objet d’une erreur d’exportation. Actualisez la page du portail et recommencez après la synchronisation suivante. Le cycle de synchronisation par défaut est de 30 minutes. 


**Q.** Que se passe-t-il si l’**objet existant** est l’objet à supprimer ?  
**A.** Si **l’objet existant** doit être supprimé, le processus n’implique pas de changement de **l’ancre source**. En général, vous pouvez résoudre ce problème à partir d’Active Directory local. 


**Q.** De quelle autorisation un utilisateur a-t-il besoin pour appliquer le correctif ?  
**A.** Le rôle **Administrateur général** ou **Contributeur** d’Azure RBAC est autorisé à accéder au processus de diagnostic et de résolution des problèmes.


**Q.** Dois-je configurer Azure AD Connect ou mettre à jour l’agent Azure AD Connect Health pour cette fonctionnalité ?  
**A.** Non, le processus de diagnostic est une fonctionnalité entièrement basée sur le cloud.


**Q.** Si l’objet existant est supprimé de manière réversible, est-ce que le processus de diagnostic rendra à l’objet son état actif ?  
**A.** Non, le correctif ne mettra pas à jour d’autres attributs d’objet qu’**Ancre source**.