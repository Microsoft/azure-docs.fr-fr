---
title: 'Azure AD Connect : Écriture différée de groupe'
description: Cet article décrit l’écriture différée de groupe dans Azure AD Connect.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.date: 06/11/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: e2e24246c749978cd2bbb5b3d0821eea6d7dfb4b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89660874"
---
# <a name="azure-ad-connect-group-writeback"></a>Écriture différée de groupe dans Azure AD Connect

Avec l’écriture différée de groupe, les clients peuvent utiliser les groupes cloud dont ils ont besoin pour leurs déploiements hybrides. Si vous utilisez la fonctionnalité Groupes Microsoft 365, ces groupes peuvent être représentés dans votre annuaire Active Directory local. Cette option est disponible **uniquement** si Exchange est présent dans votre annuaire Active Directory local.

## <a name="pre-requisites"></a>Conditions préalables
Les conditions préalables suivantes doivent être remplies pour activer la fonctionnalité d’écriture différée de groupe.
- Des licences Azure Active Directory Premium pour votre locataire.
- Un déploiement hybride configuré entre votre organisation Exchange locale et Microsoft 365, et pleinement opérationnel.
- Une version prise en charge d’Exchange local installée
- L’authentification unique configurée à l’aide d’Azure Active Directory Connect 

## <a name="enable-group-writeback"></a>Activer l’écriture différée des groupes
Pour activer l’écriture différée de groupe, effectuez les étapes suivantes :

1. Ouvrez l’Assistant Azure AD Connect, sélectionnez **Configurer**, puis cliquez sur **Suivant**.
2. Sélectionnez **Personnaliser les options de synchronisation**, puis cliquez sur **Suivant**.
3. Dans la page **Connexion à Azure AD**, entrez vos informations d’identification. Cliquez sur **Suivant**.
4. Dans la page **Fonctionnalités facultatives**, vérifiez que les options que vous avez configurées précédemment sont toujours sélectionnées.
5. Sélectionnez **Écriture différée de groupe**, puis cliquez sur **Suivant**.
6. Dans la **page Écriture**, sélectionnez une unité d’organisation Active Directory pour stocker les objets synchronisés de Microsoft 365 vers votre organisation locale, puis cliquez sur **Suivant**.
7. Dans la page **Prêt à configurer**, cliquez sur **Configurer**.
8. À la fin de l’Assistant, cliquez sur **Quitter** dans la page Configuration terminée.
9. Ouvrez Windows PowerShell en tant qu’administrateur sur le serveur Azure Active Directory Connect, puis exécutez les commandes suivantes.

```Powershell
$AzureADConnectSWritebackAccountDN =  <MSOL_ account DN>
Import-Module "C:\Program Files\Microsoft Azure Active Directory Connect\AdSyncConfig\AdSyncConfig.psm1"
Set-ADSyncUnifiedGroupWritebackPermissions -ADConnectorAccountDN $AzureADConnectSWritebackAccountDN
```

Pour plus d’informations sur la configuration des groupes Microsoft 365, consultez [Configurer des groupes Microsoft 365 dans un déploiement Exchange hybride local](/exchange/hybrid-deployment/set-up-microsoft-365-groups#enable-group-writeback-in-azure-ad-connect).

## <a name="disabling-group-writeback"></a>Désactiver l’écriture différée de groupe
Pour désactiver l’écriture différée de groupe, effectuez les étapes suivantes : 


1. Lancez l’Assistant Azure Active Directory Connect et accédez à la page Tâches supplémentaires. Sélectionnez la tâche **Personnalisation des options de synchronisation**, puis cliquez sur **Suivant**.
2. Dans la page **Fonctionnalités facultatives**, décochez l’écriture différée.  Vous recevez un avertissement vous informant que les groupes vont être supprimés.  Cliquez sur **Oui**.
   >[!IMPORTANT]
   > En désactivant l’écriture différée de groupe, les groupes créés précédemment par cette fonctionnalité seront supprimés de votre annuaire Active Directory local lors du prochain cycle de synchronisation. 

   ![Décocher la case](media/how-to-connect-group-writeback/group2.png)
  
3. Cliquez sur **Suivant**.
4. Cliquez sur **Configurer**.

 >[!NOTE]
 > La désactivation de l’écriture différée de groupe affecte la valeur « true » aux indicateurs Importation complète et Synchronisation complète sur le connecteur Azure Active Directory. Cela entraînera la propagation des modifications de règle au cours du prochain cycle de synchronisation, et la suppression des groupes précédemment réécrits dans votre annuaire Active Directory.

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur l’ [intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).