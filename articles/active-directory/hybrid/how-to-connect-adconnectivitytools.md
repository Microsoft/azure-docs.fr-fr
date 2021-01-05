---
title: 'Azure AD Connect : Présentation du module PowerShell ADConnectivityTool | Microsoft Docs'
description: Ce document présente le nouveau module PowerShell ADConnectivity et explique comment il peut vous aider à résoudre les problèmes.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 4/25/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: d7eed3e06ab42671d9674ad3893a88dfe9817e22
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95973454"
---
# <a name="troubleshoot-azure-ad-connectivity-with-the-adconnectivitytool-powershell-module"></a>Résoudre les problèmes de connectivité Azure AD avec le module PowerShell ADConnectivityTool

L’outil ADConnectivity est un module PowerShell utilisé dans l’une des situations suivantes :

- Pendant l’installation quand un problème de connectivité réseau empêche la validation des informations d’identification Active Directory fournies par l’utilisateur dans l’Assistant.
- Après l’installation par un utilisateur qui appelle les fonctions à partir d’une session PowerShell.

L’outil se trouve à l’emplacement suivant : **C:\Program Files\Microsoft Azure Active Directory Connect\Tools\ ADConnectivityTool.psm1** 

## <a name="adconnectivitytool-during-installation"></a>ADConnectivityTool pendant l’installation

Dans la page **Connexion de vos annuaires**, dans l’Assistant Azure AD Connect, si un problème réseau se produit, ADConnectivityTool utilise automatiquement l’une de ses fonctions pour déterminer ce qui se passe.  Les éléments suivants peuvent être considérés comme des problèmes réseau :

- Le nom de la forêt fourni par l’utilisateur a été mal tapé ou ladite forêt n’existe pas 
- Le port UDP 389 est fermé dans les contrôleurs de domaine associés à la forêt spécifiée par l’utilisateur
- Les informations d’identification fournies dans la fenêtre « Compte de forêt AD » ne disposent pas des privilèges nécessaires pour récupérer les contrôleurs de domaine associés à la forêt cible
- Le port TCP 53, 88 ou 389 est fermé dans les contrôleurs de domaine associés à la forêt spécifiée par l’utilisateur 
- Le port UDP 389 et un ou plusieurs ports TCP sont fermés.
- DNS n’a pas pu être résolu pour la forêt spécifiée et/ou ses contrôleurs de domaine associés

Chaque fois que l’un de ces problèmes est détecté, un message d’erreur associé s’affiche dans l’Assistant AADConnect :


![Error](media/how-to-connect-adconnectivitytools/error1.png)

Par exemple, quand nous tentons d’ajouter un annuaire sur l’écran **Connexion de vos annuaires**, Azure AD Connect doit procéder à une vérification et s’attend à pouvoir communiquer avec un contrôleur de domaine sur le port 389.  S’il ne le peut pas, l’erreur indiquée dans la capture d’écran ci-dessus s’affiche.  

En fait, Azure AD Connect appelle la fonction `Start-NetworkConnectivityDiagnosisTools` en arrière-plan.  Cette fonction est appelée quand la validation des informations d’identification échoue en raison d’un problème de connectivité réseau.

Pour finir, un fichier journal détaillé est généré chaque fois que l’outil est appelé à partir de l’Assistant. Le journal se trouve dans **C:\ProgramData\AADConnect\ADConnectivityTool-\<date>-\<time>.log**

## <a name="adconnectivitytools-post-installation"></a>ADConnectivityTools après l’installation
Après l’installation d’Azure AD Connect, toutes les fonctions du module PowerShell ADConnectivityTools peuvent être utilisées.  

Vous trouverez des informations de référence sur les fonctions dans la [Référence d’ADConnectivityTools](reference-connect-adconnectivitytools.md).

### <a name="start-connectivityvalidation"></a>Start-ConnectivityValidation

Nous allons appeler cette fonction car elle peut **uniquement** être appelée manuellement une fois qu’ADConnectivityTool.psm1 a été importé dans PowerShell. 

Cette fonction exécute la même logique que celle exécutée par l’Assistant Azure AD Connect pour valider les informations d’identification AD fournies.  Toutefois, elle procure une explication beaucoup plus détaillée concernant le problème et une suggestion de solution. 

La validation de connectivité se compose des étapes suivantes :
-   Obtenir l’objet de nom de domaine complet du domaine
-   Valider le fait que, si l’utilisateur a sélectionné « Créer un compte AD », ces informations d’identification appartiennent au groupe Administrateurs d’entreprise
-   Obtenir l’objet de nom de domaine complet de la forêt
-   Confirmer qu’au moins un domaine associé à l’objet de nom de domaine complet de la forêt obtenu précédemment est accessible
-   Vérifier que le niveau fonctionnel de la forêt est Windows Server 2003 ou plus

L’utilisateur pourra ajouter un annuaire si toutes ces actions ont été exécutées avec succès.

Si l’utilisateur exécute cette fonction après qu’un problème a été résolu (ou s’il n’y a aucun problème), la sortie invitera l’utilisateur à revenir à l’Assistant Azure AD Connect et à réessayer d’entrer les informations d’identification.



## <a name="next-steps"></a>Étapes suivantes
- [Azure AD Connect : comptes et autorisations](reference-connect-accounts-permissions.md)
- [Installation Express](how-to-connect-install-express.md)
- [Installation personnalisée](how-to-connect-install-custom.md)
- [Référence d’ADConnectivityTools](reference-connect-adconnectivitytools.md)

