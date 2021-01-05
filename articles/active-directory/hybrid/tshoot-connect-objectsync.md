---
title: 'Azure AD Connect : Résoudre les problèmes de synchronisation d’objets | Microsoft Docs'
description: Cet article explique comment résoudre les problèmes de synchronisation d’objets à l’aide de la tâche de résolution des problèmes.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: curtand
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 04/29/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 73d4239dd34f2a64aa7b3edbf88bad4348e01291
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85356200"
---
# <a name="troubleshoot-object-synchronization-with-azure-ad-connect-sync"></a>Résoudre les problèmes de synchronisation d’objets avec la synchronisation Azure AD Connect
Cet article explique les étapes de résolution des problèmes de synchronisation d’objets à l’aide de la tâche de résolution des problèmes. Pour voir le fonctionnement de la résolution des problèmes dans Azure Active Directory (Azure AD) Connect, regardez [cette courte vidéo](https://aka.ms/AADCTSVideo).

## <a name="troubleshooting-task"></a>Tâche de résolution des problèmes
Dans le cas d’un déploiement d’Azure AD Connect version 1.1.749.0 ou ultérieure, utilisez la tâche de résolution des problèmes disponible dans l’Assistant pour résoudre les problèmes de synchronisation d’objets. Pour les versions antérieures, corrigez les problèmes manuellement en suivant la procédure décrite [ici](tshoot-connect-object-not-syncing.md).

### <a name="run-the-troubleshooting-task-in-the-wizard"></a>Exécuter la tâche de résolution des problèmes de l’Assistant
Pour exécuter la tâche de résolution des problèmes de l’Assistant, procédez comme suit :

1.  Ouvrez une nouvelle session Windows PowerShell sur votre serveur Azure AD Connect avec l’option Exécuter en tant qu’administrateur.
2.  Exécutez `Set-ExecutionPolicy RemoteSigned` ou `Set-ExecutionPolicy Unrestricted`.
3.  Lancez l’Assistant Azure AD Connect.
4.  Accédez à la page Tâches supplémentaires, sélectionnez Résoudre les problèmes, puis cliquez sur Suivant.
5.  Dans la page de résolution des problèmes, cliquez sur Lancer pour ouvrir le menu de dépannage de PowerShell.
6.  Dans le menu principal, sélectionnez Troubleshoot Object Synchronization (Résoudre les problèmes de synchronisation d’objets).
![Résoudre les problèmes de synchronisation d’objets](media/tshoot-connect-objectsync/objsynch11.png)

### <a name="troubleshooting-input-parameters"></a>Paramètres d’entrée de la tâche de résolution des problèmes
La tâche de résolution des problèmes requiert les paramètres d’entrée suivants :
1.  **Nom unique de l’objet** : nom unique de l’objet auquel doit s’appliquer la résolution des problèmes.
2.  **Nom du Connecteur AD** : nom de la forêt AD où réside l’objet ci-dessus.
3.  Informations d’identification d’administrateur général pour le locataire Azure AD ![informations d’identification d’administrateur général](media/tshoot-connect-objectsync/objsynch1.png)

### <a name="understand-the-results-of-the-troubleshooting-task"></a>Comprendre les résultats de la tâche de résolution des problèmes
La tâche de résolution des problèmes effectue les vérifications suivantes :

1.  détection d’une incompatibilité de nom d’utilisateur principal (UPN) si l’objet est synchronisé avec Azure Active Directory ;
2.  vérification si l’objet a été exclu en raison d’un filtrage de domaine ;
3.  vérification si l’objet a été exclu en raison d’un filtrage d’unité d’organisation.
4.  Vérifiez si la synchronisation des objets est bloquée en raison d’une boîte aux lettres liée
5. Vérifiez si l’objet est un groupe de distribution dynamique qui n’est pas censé être synchronisé

Le reste de cette section décrit les résultats spécifiques qui sont renvoyés par la tâche. Dans chaque cas, la tâche fournit une analyse suivie des actions recommandées pour résoudre le problème.

## <a name="detect-upn-mismatch-if-object-is-synced-to-azure-active-directory"></a>Détection d’une incompatibilité de nom d’utilisateur principal (UPN) si l’objet est synchronisé avec Azure Active Directory
### <a name="upn-suffix-is-not-verified-with-azure-ad-tenant"></a>Suffixe UPN NON vérifié auprès du locataire Azure AD
Lorsque le suffixe d’UPN/ID de connexion alternatif n’est pas vérifié auprès du locataire Azure AD, Azure Active Directory remplace les suffixes UPN par le nom de domaine par défaut « onmicrosoft.com ».

![Azure AD remplace UPN](media/tshoot-connect-objectsync/objsynch2.png)

### <a name="azure-ad-tenant-dirsync-feature-synchronizeupnformanagedusers-is-disabled"></a>Fonctionnalité DirSync « SynchronizeUpnForManagedUsers » du locataire Azure AD désactivée
Lorsque la fonctionnalité DirSync « SynchronizeUpnForManagedUsers » du locataire Azure AD est désactivée, Azure Active Directory n’autorise pas les mises à jour de synchronisation d’UPN/ID de connexion alternatif pour les comptes d’utilisateur sous licence avec authentification gérée.

![SynchronizeUpnForManagedUsers](media/tshoot-connect-objectsync/objsynch4.png)

## <a name="object-is-filtered-due-to-domain-filtering"></a>Objet exclu en raison d’un filtrage de domaine
### <a name="domain-is-not-configured-to-sync"></a>Domaine non configuré pour la synchronisation
L’objet n’entre pas dans le champ d’application de la synchronisation, car le domaine n’est pas configuré. Dans l’exemple ci-après, la synchronisation ne s’applique pas à l’objet, car le domaine auquel appartient ce dernier a été exclu de la synchronisation.

![Domaine non configuré pour la synchronisation](media/tshoot-connect-objectsync/objsynch5.png)

### <a name="domain-is-configured-to-sync-but-is-missing-run-profilesrun-steps"></a>Domaine configuré pour la synchronisation, mais dépourvu de profils d’exécution/étapes d’exécution
L’objet n’entre pas dans le champ d’application de la synchronisation, car le domaine ne présente pas de profils d’exécution/étapes d’exécution. Dans l’exemple ci-après, la synchronisation ne s’applique pas à l’objet, car le domaine auquel appartient ce dernier est dépourvu d’étapes d’exécution pour le profil d’exécution d’importation intégrale.
![profils d’exécution manquants](media/tshoot-connect-objectsync/objsynch6.png)

## <a name="object-is-filtered-due-to-ou-filtering"></a>Objet exclu en raison d’un filtrage d’unité d’organisation
L’objet n’entre pas dans le champ d’application de la synchronisation à cause de la configuration du filtrage d’unité d’organisation. Dans l’exemple ci-après, l’objet appartient à OU=NoSync,DC=bvtadwbackdc,DC=com.  Cette unité d’organisation n’entre pas dans le champ d’application de la synchronisation.</br>

![Unité d’organisation](./media/tshoot-connect-objectsync/objsynch7.png)

## <a name="linked-mailbox-issue"></a>Problème de boîte aux lettres liée
Une boîte aux lettres liée est censée être associée à un compte de référence externe principal situé dans une autre forêt de comptes approuvés. S’il n’y a aucun compte principal externe, Azure AD Connect ne synchronise pas le compte utilisateur correspondant à la boîte aux lettres liée dans la forêt Exchange sur le client Azure AD.</br>
![Boîte aux lettres liée](./media/tshoot-connect-objectsync/objsynch12.png)

## <a name="dynamic-distribution-group-issue"></a>Problème de groupe de distribution dynamique
En raison de plusieurs différences entre Active Directory local et Azure Active Directory, Azure AD Connect ne synchronise pas les groupes de distribution dynamiques pour le client Azure AD.

![Groupe de distribution dynamique](./media/tshoot-connect-objectsync/objsynch13.png)

## <a name="html-report"></a>Rapport HTML
Outre l’analyse de l’objet, la tâche de résolution des problèmes génère un rapport HTML incluant toutes les connaissances concernant l’objet. Ce rapport HTML peut être partagé avec l’équipe du support technique à des fins de résolution des problèmes approfondie s’il y a lieu.

![Rapport HTML](media/tshoot-connect-objectsync/objsynch8.png)

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur l’ [intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).
