---
title: Résoudre les problèmes de données d’audit lors d’un changement de domaine vérifié | Microsoft Docs
description: Présente les informations qui s’afficheront dans les journaux d’activité Azure Active Directory si vous changez un domaine utilisateur vérifié.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 07/22/2020
ms.author: markvi
ms.collection: M365-identity-device-management
ms.openlocfilehash: f3c9ec3b1e96e47dbf46c6acb2c81147b614d069
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87117413"
---
# <a name="troubleshoot-audit-data-on-verified-domain-change"></a>Résoudre les problèmes : Auditer les données après un changement de domaine vérifié 


## <a name="i-have-a-lot-of-changes-to-my-users-and-i-am-not-sure-what-the-cause-of-it-is"></a>Mes utilisateurs ont fait l’objet de nombreuses modifications et je ne suis pas certain d’en connaître la cause.

### <a name="symptoms"></a>Symptômes

Je consulte les journaux d’audit Azure AD et j’observe plusieurs mises à jour d’utilisateur dans mon locataire Azure AD. Ces événements **Mise à jour d’utilisateur** ne présentent pas les informations **Acteur**. Il est donc difficile de déterminer l’action/la personne qui a déclenché les modifications de masse sur les utilisateurs. 

### <a name="cause"></a>Cause

 L’opération back-end non synchrone **ProxyCalc** est une cause courante de modifications d’objet de masse.  **ProxyCalc** est la logique qui détermine le **UserPrincipalName** et les **adresses proxy** appropriés, qui sont mis à jour pour les utilisateurs, groupes ou contacts Azure AD. **ProxyCalc** est conçu pour garantir que tous les **UserPrincipalName** et **adresses proxy** sont cohérents à tout moment dans Azure AD. **ProxyCalc** doit être déclenché par un changement explicite comme un changement de domaine vérifié. Il ne s’exécute pas en permanence en arrière-plan comme tâche. 

  

#### <a name="what-does-userprincipalname-consistency-mean"></a>Qu’est-ce que la cohérence du UserPrincipalName ? 

Pour les utilisateurs cloud uniquement, la cohérence signifie que le **UserPrincipalName** est défini sur un suffixe de domaine vérifié. Quand un **UserPrincipalName** incohérent est traité, **ProxyCalc** le convertit avec le suffixe onmicrosoft.com par défaut, par exemple : username@Contoso.onmicrosoft.com 

Pour les utilisateurs synchronisés, la cohérence signifie que le **UserPrincipalName** est défini sur un suffixe de domaine vérifié et correspond à la valeur du **UserPrincipalName** local (ShadowUserPrincipalName). Quand un **UserPrincipalName** incohérent est traité, **ProxyCalc** rétablit la valeur du **ShadowUserPrincipalName** ou, si ce suffixe de domaine a été supprimé du locataire, le convertit avec le suffixe de domaine *.onmicrosoft.com par défaut. 

  

#### <a name="what-does-proxy-address-consistency-mean"></a>Qu’est-ce que la cohérence des adresses proxy ? 

Pour les utilisateurs cloud uniquement, la cohérence signifie que les adresses proxy correspondent à un suffixe de domaine vérifié. Quand une adresse proxy incohérente est traitée, **ProxyCalc** la convertit avec le suffixe de domaine *.onmicrosoft.com par défaut, par exemple : SMTP:username@Contoso.onmicrosoft.com 

Pour les utilisateurs synchronisés, la cohérence signifie que les adresses proxy correspondent à la valeur de l’adresse ou des adresses proxy locales (ShadowProxyAddresses). Les **adresses proxy** sont censées être synchronisées avec les **ShadowProxyAddresses**. Si une licence Exchange est attribuée à l’utilisateur synchronisé, les adresses proxy doivent correspondre à la valeur de l’adresse ou des adresses proxy locales et doivent également correspondre à un suffixe de domaine vérifié. Dans ce scénario, **ProxyCalc** nettoie l’adresse proxy incohérente (avec le suffixe de domaine non vérifié) et la supprime de l’objet dans Azure AD. Si ce domaine non vérifié est vérifié ultérieurement, **ProxyCalc** recalcule et ajoute de nouveau l’adresse proxy à l’objet dans Azure AD à partir des **ShadowProxyAddresses**.  

> [!NOTE]
> Pour les objets synchronisés, afin d’éviter que la logique **ProxyCalc** ne calcule des résultats inattendus, il est préférable de définir des adresses proxy sur un domaine Azure AD vérifié sur l’objet local.  

  
Chaque changement de domaine vérifié est une tâche d’administration pouvant déclencher **ProxyCalc**. Cette tâche se produit chaque fois qu’un domaine vérifié est ajouté à un locataire Azure AD ou en est supprimé, ce qui déclenche **ProxyCalc** en interne.  

Par exemple, si vous ajoutez le domaine vérifié Fabrikam.com à votre locataire Contoso.onmicrosoft.com, cette action déclenche une opération ProxyCalc sur tous les objets du locataire. Cet événement est capturé dans les journaux d’audit Azure AD comme événement **Mise à jour d’utilisateur**, précédé d’un événement **Ajout d’un domaine vérifié**. En revanche, si Fabrikam.com a été supprimé du locataire Contoso.onmicrosoft.com, tous les événements **Mise à jour d’utilisateur** sont précédés d’un événement **Suppression de domaine vérifié**.   

#### <a name="additional-notes"></a>Informations complémentaires :

ProxyCalc n’entraîne pas la modification de certains objets qui : 

- N’ont pas de licence Exchange active. 
- Ont **MSExchRemoteRecipientType** défini sur Null. 
- Ne sont pas considérés comme ressources partagées. Un objet est considéré comme ressource partagée quand **CloudMSExchRecipientDisplayType** contient l’une des valeurs suivantes : **MailboxUser (shared)** , **PublicFolder**, **ConferenceRoomMailbox**, **EquipmentMailbox**, **ArbitrationMailbox**, **RoomList**, **TeamMailboxUser**, **Group mailbox**, **Scheduling mailbox**, **ACLableMailboxUser**, **ACLableTeamMailboxUser** 
  
 Pour établir une meilleure corrélation entre ces deux événements disparates, Microsoft travaille à la mise à jour de l’information **Acteur** dans les journaux d’audit pour identifier ces modifications comme déclenchées par un changement de domaine vérifié. Cette action permet de vérifier à quel moment l’événement de changement de domaine vérifié s’est produit et a lancé la mise à jour de masse des objets dans leur locataire. 

De plus, dans la plupart des cas, les utilisateurs ne font l’objet d’aucune modification, car leur **UserPrincipalName** et leurs **adresses de proxy** sont cohérents. L’amélioration sur laquelle nous travaillons actuellement consiste à afficher dans les journaux d’audit uniquement les mises à jour qui ont entraîné une modification réelle de l’objet. Cette action supprimera des informations parasite dans les journaux d’audit et permettra aux administrateurs de mettre en corrélation les autres modifications d’utilisateurs avec l’événement de changement de domaine vérifié comme expliqué ci-dessus. 

## <a name="next-steps"></a>Étapes suivantes

[Attributs de clichés instantanés du service de synchronisation Azure AD Connect](../hybrid/how-to-connect-syncservice-shadow-attributes.md)
