---
title: Résoudre les problèmes liés aux appartenances de groupes dynamiques - Azure AD | Microsoft Docs
description: Conseils de résolution des problèmes pour l’appartenance à un groupe dynamique dans Azure Active Directory
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: enterprise-users
ms.topic: troubleshooting
ms.date: 12/02/2020
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 41bfdf11bad28ab772b68839a5a7bf7776eb4dff
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96548101"
---
# <a name="troubleshoot-and-resolve-groups-issues"></a>Dépanner et résoudre des problèmes de groupes

## <a name="troubleshooting-group-creation-issues"></a>Résolution de problèmes de création de groupe

**J’ai désactivé la création de groupe de sécurité dans le portail Azure, mais il est toujours possible de créer des groupes via Powershell** Le paramètre **Les utilisateurs peuvent créer des groupes de sécurité dans les portails Azure** sur le portail Azure contrôle la possibilité ou non pour des utilisateurs non-administrateurs de créer des groupes de sécurité dans le Panneau d’accès ou le portail Azure. Il ne contrôle pas la création de groupe de sécurité via Powershell.

Pour désactiver la création de groupe par des utilisateurs non-administrateurs :
1. Vérifiez que les utilisateurs non-administrateurs sont autorisés à créer des groupes :
   

   ```powershell
   Get-MsolCompanyInformation | Format-List UsersPermissionToCreateGroupsEnabled
   ```

  
2. Si la commande retourne `UsersPermissionToCreateGroupsEnabled : True`, les utilisateurs non-administrateurs peuvent créer des groupes. Pour désactiver cette fonctionnalité :
  

   ``` 
   Set-MsolCompanySettings -UsersPermissionToCreateGroupsEnabled $False
   ```

<br/>**En tentant de créer un groupe dynamique dans Powershell, j’ai reçu un message d’erreur me signalant que le nombre maximal de groupes autorisé était atteint**<br/>
Si vous recevez dans Powershell le message d’erreur _le nombre maximal de groupes a été atteint dans les stratégies de groupe dynamiques_, cela signifie que vous avez atteint la limite maximale du nombre de groupes dynamiques que votre organisation peut compter. Le nombre maximal de groupes dynamiques par organisation est de 5 000.

Pour créer de nouveaux groupes dynamiques, vous devez commencer par supprimer des groupes dynamiques existants. Il n’existe aucun moyen d’augmenter cette limite.

## <a name="troubleshooting-dynamic-memberships-for-groups"></a>Résolution des problèmes liés à l’appartenance dynamique à des groupes

**J’ai configuré une règle sur un groupe, mais aucune appartenance n’est mise à jour dans le groupe**<br/>
1. Vérifiez les valeurs d’attributs d’utilisateur ou d’appareil dans la règle. Vérifiez que les utilisateurs satisfont à la règle. Pour les appareils, vérifiez leurs propriétés pour vous assurer que les attributs synchronisés contiennent les valeurs attendues.<br/>
2. Vérifiez l’état du traitement de l’appartenance pour vous assurer que le traitement est terminé. Vous pouvez voir l’**État du traitement de l’appartenance** et la date de la dernière mise à jour dans la page [Vue d’ensemble](groups-create-rule.md#check-processing-status-for-a-rule) du groupe.

Si tout semble correct, attendez quelque temps avant que le groupe se remplisse. Selon la taille de votre organisation Azure AD, le remplissage du groupe peut prendre jusqu’à 24 heures la première fois ou après une modification de la règle.

**J’ai configuré une règle, mais les membres existants sur celle-ci sont à présent supprimés**<br/>Ce comportement est normal. Lors de l’activation ou de la modification d’une règle, les membres existants du groupe sont supprimés. Les utilisateurs provenant de l’évaluation de la règle sont ajoutés en tant que membres du groupe.

**Je ne vois pas instantanément de changement d’appartenance quand j’ajoute ou quand je modifie une règle. Pourquoi ?**<br/>L’évaluation de l’appartenance dédiée est effectuée régulièrement dans le cadre d’un processus asynchrone exécuté en arrière-plan. La durée de l’opération est déterminée par le nombre d’utilisateurs dans votre annuaire et par la taille du groupe créé à la suite de la règle. Généralement, les annuaires contenant peu d’utilisateurs constatent les changements d’appartenance au groupe en moins de quelques minutes. Le renseignement des annuaires contenant de nombreux utilisateurs peut prendre plus de 30 minutes.

**Comment puis-je forcer le traitement immédiat du groupe ?**<br/>
Actuellement, il n’existe aucun moyen de déclencher automatiquement le traitement du groupe à la demande. En revanche, vous pouvez déclencher manuellement son retraitement en mettant à jour la règle d’appartenance par l’ajout d’une espace à la fin de celle-ci.  

**J’ai rencontré une erreur de traitement de règle**<br/>Le tableau suivant répertorie les erreurs de règle d’appartenance dynamique courantes et la façon de les corriger.

| Erreur d’analyse de la règle | Utilisation incorrecte | Utilisation corrigée |
| --- | --- | --- |
| Erreur : attribut non pris en charge. |(user.invalidProperty -eq "Value") |(user.department -eq "value")<br/><br/>Assurez-vous que l’attribut se trouve dans la [liste des propriétés prises en charge](groups-dynamic-membership.md#supported-properties). |
| Erreur : l’opérateur n’est pas pris en charge sur l’attribut. |(user.accountEnabled -contains true) |(user.accountEnabled - eq true)<br/><br/>L’opérateur utilisé n’est pas pris en charge pour le type de propriété (dans cet exemple,-contains ne peut pas être utilisé avec le type booléen). Utilisez les opérateurs corrects pour le type de propriété. |
| Erreur : erreur de compilation de la requête. | 1. (user.department -eq "Sales") (user.department -eq "Marketing")<br>2. (user.userPrincipalName -match "*@domain.ext") | 1. Opérateur manquant. Utilisez -and ou -or pour associer les prédicats<br>(user.department -eq "Sales") -or (user.department -eq "Marketing")<br>2. Erreur dans l’expression régulière utilisée avec -match<br>(user.userPrincipalName -match ".*@domain.ext")<br>ou bien : (user.userPrincipalName -match "@domain.ext$") |

## <a name="next-steps"></a>Étapes suivantes

Ces articles fournissent des informations supplémentaires sur Azure Active Directory.

* [Gestion de l’accès aux ressources avec les groupes Azure Active Directory](../fundamentals/active-directory-manage-groups.md)
* [Gestion des applications dans Azure Active Directory](../manage-apps/what-is-application-management.md)
* [Qu’est-ce qu’Azure Active Directory ?](../fundamentals/active-directory-whatis.md)
* [Intégration des identités locales dans Azure Active Directory](../hybrid/whatis-hybrid-identity.md)