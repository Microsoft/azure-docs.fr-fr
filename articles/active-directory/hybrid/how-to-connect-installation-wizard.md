---
title: Réexécution de l’Assistant Installation d’Azure AD Connect | Microsoft Docs
description: Explique le fonctionnement de l’Assistant Installation la deuxième fois que vous l’exécutez.
keywords: L’Assistant Installation d’Azure AD Connect vous permet de configurer les paramètres de maintenance la deuxième fois que vous l’exécutez
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: d800214e-e591-4297-b9b5-d0b1581cc36a
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/17/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9d81836b47acb19f624075480aafef74c9c0934c
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91306121"
---
# <a name="azure-ad-connect-sync-running-the-installation-wizard-a-second-time"></a>Azure AD Connect Sync : exécuter l’Assistant Installation une deuxième fois
La première fois que vous exécutez l’Assistant Installation d’Azure AD Connect, il vous guide dans la procédure de configuration de votre installation. Si vous réexécutez l’Assistant Installation, il vous propose des options de maintenance.

>[!IMPORTANT]
>N’oubliez pas que vous ne pouvez pas exécuter l’Assistant Installation pendant qu’une synchronisation est en cours.  Avant de lancer l’Assistant, vérifiez qu’aucune synchronisation n’est en cours d’exécution.

Vous trouverez l’Assistant Installation dans le menu Démarrer sous le nom **Azure Connect AD**.

![Menu Démarrer](./media/how-to-connect-installation-wizard/startmenu.png)

Lorsque vous démarrez l’Assistant Installation, une page s’affiche avec ces options :

![Page avec une liste de tâches supplémentaires](./media/how-to-connect-installation-wizard/additionaltasks.png)

Si vous avez installé AD FS avec Azure AD Connect, vous avez davantage d’options. Les options supplémentaires que vous avez pour AD FS sont documentées dans [Gestion AD FS](how-to-connect-fed-management.md#manage-ad-fs).

Sélectionnez l’une des tâches et cliquez sur **Suivant** pour continuer.

> [!IMPORTANT]
> Tant que l’Assistant Installation est ouvert, toutes les opérations du moteur de synchronisation sont suspendues. Prenez soin de fermer l’Assistant Installation dès que vous avez terminé vos modifications de configuration.
>
>

## <a name="view-current-configuration"></a>Afficher la configuration actuelle
Cette option vous donne un aperçu rapide de vos options actuellement configurées.

![Page avec la liste de toutes les options et leur état](./media/how-to-connect-installation-wizard/viewconfig.png)

Cliquez sur **Précédent** pour revenir en arrière. Si vous sélectionnez **Quitter**, vous fermez l’Assistant Installation.

## <a name="customize-synchronization-options"></a>Personnaliser les options de synchronisation
Cette option est utilisée pour apporter des modifications à la configuration de la synchronisation. Vous voyez un sous-ensemble d’options provenant du chemin d’installation de la configuration personnalisée. Et ce, même si vous avez initialement utilisé l’installation rapide.

* [Ajouter d’autres annuaires](how-to-connect-install-custom.md#connect-your-directories). Pour supprimer un annuaire, consultez [Supprimer un connecteur](how-to-connect-sync-service-manager-ui-connectors.md#delete).
* [Modifier le filtrage par domaine ou unité d’organisation](how-to-connect-install-custom.md#domain-and-ou-filtering).
* Supprimer le filtrage de groupe.
* [Modifier les fonctionnalités facultatives](how-to-connect-install-custom.md#optional-features).

Les autres options provenant de l’installation initiale ne peuvent pas être modifiées et ne sont pas disponibles. Ces options sont :

* Modifier l’attribut à utiliser pour userPrincipalName et sourceAnchor.
* Modifier la méthode de jointure d’objets provenant de différentes forêts.
* Activer le filtrage de groupe.

## <a name="refresh-directory-schema"></a>Actualiser le schéma d’annuaire
Cette option est utilisée si vous avez modifié le schéma dans l’une de vos forêts AD DS locales. Par exemple, vous pouvez avoir installé Exchange ou effectué une mise à niveau vers un schéma Windows Server 2012 avec des objets de périphérique. Dans ce cas, vous devez demander à Azure AD Connect de relire le schéma AD DS et de mettre à jour son cache. Cette action régénère également les règles de synchronisation. Si vous ajoutez le schéma Exchange, par exemple, les règles de synchronisation d’Exchange sont ajoutées à la configuration.

Lorsque vous sélectionnez cette option, tous les annuaires de votre configuration sont répertoriés. Vous pouvez conserver le paramètre par défaut et actualiser toutes les forêts ou désélectionner certaines d’entre elles.

![Page avec la liste de tous les annuaires de l’environnement](./media/how-to-connect-installation-wizard/refreshschema.png)

## <a name="configure-staging-mode"></a>Configurer le mode de préproduction
Cette option permet d’activer et de désactiver le mode de préproduction sur le serveur. Vous trouverez plus d’informations sur le mode de préproduction et son utilisation dans [Opérations](how-to-connect-sync-staging-server.md).

L’option s’affiche si la préproduction est actuellement activée ou désactivée :  
![Capture d’écran montrant le mode intermédiaire désactivé.](./media/how-to-connect-installation-wizard/stagingmodecurrentstate.png)

Pour modifier l’état, sélectionnez cette option et cochez ou décochez la case.  
![Option qui affiche également l’état actuel du mode de préproduction](./media/how-to-connect-installation-wizard/stagingmodeenable.png)

## <a name="change-user-sign-in"></a>Modifier la connexion de l’utilisateur
Cette option permet de définir la méthode de connexion utilisateur sur la synchronisation de hachage de mot de passe, l’authentification directe ou la fédération. Vous ne pouvez pas passer à **ne pas configurer**.

Pour plus d’informations sur cette option, consultez [connexion de l’utilisateur](plan-connect-user-signin.md#changing-the-user-sign-in-method).

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur le modèle de configuration utilisé par la synchronisation d’Azure AD Connect dans [Comprendre l’approvisionnement déclaratif](concept-azure-ad-connect-sync-declarative-provisioning.md).

**Rubriques de présentation**

* [Azure AD Connect Sync - Présentation et personnalisation des options de synchronisation](how-to-connect-sync-whatis.md)
* [Intégration des identités locales dans Azure Active Directory](whatis-hybrid-identity.md)
