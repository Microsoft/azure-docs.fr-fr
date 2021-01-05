---
title: Utilisateurs avec indicateur de risque dans le portail Azure Active Directory | Microsoft Docs
description: En savoir plus sur le rapport sur la sécurité des utilisateurs avec indicateur de risque dans le portail Azure Active Directory
services: active-directory
author: MarkusVi
manager: daveba
ms.assetid: addd60fe-d5ac-4b8b-983c-0736c80ace02
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 11/13/2018
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 23bd7a26bc6215f50a2860ab4aac84f1b46301f5
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92308940"
---
# <a name="remediate-users-flagged-for-risk-in-the-azure-active-directory-portal"></a>Résoudre des utilisateurs avec indicateur de risque dans le portail Azure Active Directory

Grâce aux rapports sur la sécurité dans Azure Active Directory (Azure AD), vous pouvez évaluer les risques de compromission des comptes d’utilisateur au sein de votre environnement. Un utilisateur signalé comme présentant un risque indique qu’un compte d’utilisateur est susceptible d’avoir été compromis.

Microsoft s’engage à vous aider à conserver un environnement protégé. Dans le cadre de cet engagement, Microsoft surveille continuellement les activités inhabituelles ou qui correspondent à des modèles d’attaque connus. 

Si des activités inhabituelles indiquant un accès non autorisé à des comptes de vos utilisateurs sont détectées, vous recevez des notifications qui vous permettent d’agir. Cela ne signifie pas que les systèmes Microsoft ont été compromis.

## <a name="access-the-users-flagged-for-risk-report"></a>Accéder au rapport des utilisateurs avec indicateur de risque

Vous pouvez analyser les utilisateurs signalés comme présentant un risque par le biais du [rapport sur les utilisateurs à risque](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RiskyUsers) dans le portail Azure. Si vous n'avez pas Azure AD, vous pouvez vous inscrire gratuitement sur [https://aka.ms/AccessAAD](https://aka.ms/AccessAAD). 

À partir du rapport sur les utilisateurs signalés comme présentant un risque, vous pouvez effectuer les actions suivantes pour chaque utilisateur :

- Générer un mot de passe temporaire
- Demander à l’utilisateur de réinitialiser son mot de passe la prochaine fois qu’il se connecte
- Supprimer le risque de l’utilisateur sans aucune action de correction.

Pour plus d’informations, consultez [Rapport de sécurité des utilisateurs associés à un indicateur de risque](../identity-protection/overview-identity-protection.md).

### <a name="azure-ad-subscription-for-microsoft-365-customers"></a>Abonnement Azure AD de clients Microsoft 365

Vous pouvez également utiliser vos informations d’identification Microsoft 365 pour accéder au **centre d’administration Azure**. Lorsque votre accès à Azure AD est activé, vous êtes redirigé vers le portail Azure AD. Au niveau d’inscription de base, la quantité d’informations fournies dans les rapports est limitée. Des analyses et données supplémentaires sont disponibles pour les abonnés Azure Premium.

Pour accéder aux rapports sur les **utilisateurs avec indicateur de risque** dans le centre d’administration Microsoft 365 :

1.  À gauche du menu de navigation, sélectionnez **Centres d’administration**. 
2.  Sélectionnez ensuite **Azure AD**.
3.  Connectez-vous au **centre d’administration Azure Active Directory**.
4.  Si une bannière s’affiche en haut de la page, indiquant **Vérifier le nouveau portail**, sélectionnez le lien.
4.  Dans le menu de navigation situé à gauche, sélectionnez **Azure Active Directory**. 
5.  Dans le volet de navigation, sélectionnez **Utilisateurs associés à un indicateur de risque** depuis la section **Sécurité**.

## <a name="remediation-actions"></a>Actions de correction

Effectuez les actions suivantes pour aider à rectifier les comptes affectés et à sécuriser votre environnement :

1.  [Validez les informations correctes](https://aka.ms/MFAValid) pour l’authentification multifacteur et la réinitialisation de mot de passe libre-service. 
2.  [Activez l’authentification multifacteur](/microsoft-365/admin/security-and-compliance/set-up-multi-factor-authentication) pour tous les utilisateurs. 
3.  Utilisez ce [script de correction](https://aka.ms/remediate) pour chaque compte affecté afin d'effectuer les étapes suivantes : 

    a. Réinitialisez le mot de passe pour sécuriser le compte et éliminez les sessions actives.

    b. Supprimez les délégués de boîte aux lettres.

    c. Désactivez les règles d’envoi de courrier à des domaines externes.

    d. Supprimez la propriété d’envoi de courrier global de la boîte aux lettres.

    e. Activez l’authentification multifacteur (MFA) du compte de l’utilisateur.

    f. Définissez la complexité du mot de passe du compte sur « élevée ».

    g. Activez l’audit de la boîte aux lettres.

    h. Générez un journal d’audit pour permettre à l'administrateur de l’analyser.

4. Dans votre locataire Microsoft 365 et autres infrastructure informatique, analysez tous les paramètres d’abonné, comptes d’utilisateur et paramètres de configuration par utilisateur et cherchez toute modification éventuelle. Vérifiez les indicateurs de méthodes ou de persistance, ainsi que tout signe qu’un intrus aurait pu mettre la main sur des informations d’identification VPN, ou accéder à d’autres ressources organisationnelles. 

5.  Lors de votre enquête, vous devrez décider si vous devez avertir les autorités gouvernementales, concernant les lois en vigueur.

De plus, vous devez :

- Lire et implémenter ce [conseil sur la gestion d’activités inhabituelles](/archive/blogs/office365security/how-to-fix-a-compromised-hacked-microsoft-office-365-account). 
- [Activer le pipeline d’audit](/archive/blogs/office365security/using-office-365-activity-data-to-improve-your-cybersecurity-stance-and-capability) pour vous aider à analyser l’activité sur votre abonnement. Une fois fini, votre banque d’informations d’audit commence à remplir les journaux d’activités. À ce stade, vous pouvez également tirer parti de la [ressource de recherche et d'enquête du centre de sécurité et de conformité](https://aka.ms/sccsearch). 
- Vérifiez les autorisations des délégués et les règles d’envoi de courrier pour toutes vos boîtes aux lettres. Vous pouvez utiliser ce [script PowerShell](https://aka.ms/delegateforwardrules) pour réaliser cette tâche. 

## <a name="next-steps"></a>Étapes suivantes

* [Azure Active Directory Identity Protection](../identity-protection/overview-identity-protection.md)
* [Utilisateurs associés à un indicateur de risque](../identity-protection/overview-identity-protection.md)