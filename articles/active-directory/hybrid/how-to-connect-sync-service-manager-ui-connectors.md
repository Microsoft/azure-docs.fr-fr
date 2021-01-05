---
title: Connecteurs dans l’interface utilisateur d’Azure AD Synchronization Service Manager | Microsoft Docs
description: Comprendre l’onglet Connecteurs dans Synchronization Service Manager pour Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 60f1d979-8e6d-4460-aaab-747fffedfc1e
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/13/2017
ms.subservice: hybrid
ms.author: billmath
ms.custom: H1Hack27Feb2017
ms.collection: M365-identity-device-management
ms.openlocfilehash: d215c2e200308664f24daa28a0054c8f1bcfc09c
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91319875"
---
# <a name="using-connectors-with-the-azure-ad-connect-sync-service-manager"></a>Utilisation de connecteurs avec Azure AD Connect Sync Service Manager

![Capture d’écran montrant Sync Service Manager.](./media/how-to-connect-sync-service-manager-ui-connectors/connectors.png)

L’onglet Connecteurs permet de gérer tous les systèmes auquel le moteur de synchronisation est connecté.

## <a name="connector-actions"></a>Actions du connecteur
| Action | Commentaire |
| --- | --- |
| Créer |Ne pas utiliser. Pour la connexion à des forêts Active Directory supplémentaires, utilisez l’Assistant Installation. |
| Propriétés |Permet le filtrage de domaine et d’unité organisationnelle. |
| [Supprimer](#delete) |Permet de supprimer les données dans l’espace connecteur ou de supprimer la connexion à une forêt. |
| [Configurer les profils d’exécution](#configure-run-profiles) |À l’exception du filtrage de domaine, il n’y a rien à configurer ici. Vous pouvez vous servir de cette action pour voir les profils d’exécution déjà configurés. |
| Exécuter |Permet de lancer l’exécution unique d’un profil. |
| Arrêter |Arrête un connecteur qui exécute un profil. |
| Exporter le connecteur |Ne pas utiliser. |
| Importer le connecteur |Ne pas utiliser. |
| Mettre à jour le connecteur |Ne pas utiliser. |
| Actualiser le schéma |Actualise le schéma mis en cache. Il est préférable d’utiliser l’option dans l’Assistant Installation, car les règles de synchronisation sont également mises à jour. |
| [Espace de connecteur de recherche](#search-connector-space) |Permet de rechercher des objets et de Suivre un objet et ses données dans le système. |

### <a name="delete"></a>DELETE
L’action de suppression est utilisée dans deux cas.  
![Capture d’écran montrant la fenêtre « Supprimer le connecteur » avec l’option « Supprimer l’espace connecteur uniquement » activée.](./media/how-to-connect-sync-service-manager-ui-connectors/connectordelete.png)

L’option **Supprimer l’espace connecteur uniquement** supprime toutes les données en conservant la configuration.

L’option **Supprimer le connecteur et l’espace connecteur** supprime les données et la configuration. Utilisez cette option quand vous ne souhaitez plus vous connecter à une forêt.

Les deux options synchronisent tous les objets et mettent à jour les objets du métaverse. Cette action est une opération de longue durée.

### <a name="configure-run-profiles"></a>Configurer les profils d’exécution
Cette option vous permet de voir les profils d'exécution configurées pour un connecteur.

![Capture d’écran montrant la fenêtre « Configurer les profils d’exécution » avec l’option « Importation d’écart » activée.](./media/how-to-connect-sync-service-manager-ui-connectors/configurerunprofiles.png)

### <a name="search-connector-space"></a>Espace de connecteur de recherche
L’action de recherche dans l’espace connecteur permet de rechercher des objets et de résoudre les problèmes relatifs aux données.

![Capture d’écran montrant la fenêtre « Espace de connecteur de recherche ».](./media/how-to-connect-sync-service-manager-ui-connectors/cssearch.png)

Commencez par sélectionner une **portée**. Vous pouvez rechercher des données (nom unique relatif, nom de domaine, ancre, sous-arborescence) ou l’état de l’objet (toutes les autres options).  
![Capture d’écran montrant le menu déroulant « Étendue ».](./media/how-to-connect-sync-service-manager-ui-connectors/cssearchscope.png)  
Par exemple, si vous effectuez une recherche dans la sous-arborescence, vous obtenez tous les objets d’une unité d’organisation.  
![Capture d’écran montrant un exemple de recherche d’une « sous-arborescence ».](./media/how-to-connect-sync-service-manager-ui-connectors/cssearchsubtree.png)  
À partir de cette grille, vous pouvez sélectionner un objet, sélectionner des **propriétés** et [les suivre](tshoot-connect-object-not-syncing.md) par le biais du métaverse, de l’espace connecteur source vers l’espace connecteur cible.

### <a name="changing-the-ad-ds-account-password"></a>Modifier le mot de passe du compte AD DS
Si vous modifiez le mot de passe du compte, le service de synchronisation ne peut plus importer/exporter des modifications vers le répertoire Active Directory local.   Le message suivant peut apparaître :

- L’étape d’importation/exportation pour le connecteur AD échoue avec l’erreur « no-start-credentials ».
- Dans l’Observateur d’événements Windows, le journal d’événement d’application contient une erreur avec l’ID d’événement 6000 et le message « Échec de l'exécution de l'agent de gestion « contoso.com » en raison d'informations d'identification non valides ».

Pour résoudre ce problème, mettez à jour le compte d’utilisateur AD DS en procédant comme suit :


1. Démarrez Synchronization Service Manager (DÉMARRER → Service de synchronisation).
</br>![Sync Service Manager](./media/how-to-connect-sync-service-manager-ui-connectors/startmenu.png)
2. Accédez à l’onglet **Connecteurs** .
3. Sélectionnez le connecteur Active Directory configuré pour utiliser le compte AD DS.
4. Sous Actions, sélectionnez **Propriétés**.
5. Dans la boîte de dialogue contextuelle, sélectionnez Se connecter à la forêt Active Directory :
6. Le nom de la forêt indique l’AD local correspondant.
7. Le nom d’utilisateur indique le compte AD DS utilisé pour la synchronisation.
8. Entrez le nouveau mot de passe du compte AD DS dans la zone de texte Mot de passe ![Utilitaire de clé de chiffrement Azure AD Connect Sync](./media/how-to-connect-sync-service-manager-ui-connectors/key6.png)
9. Cliquez sur OK pour enregistrer le nouveau mot de passe et redémarrez le service de synchronisation pour supprimer l’ancien mot de passe du cache mémoire.



## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur la configuration de la [synchronisation Azure AD Connect](how-to-connect-sync-whatis.md) .

En savoir plus sur l’ [intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).
