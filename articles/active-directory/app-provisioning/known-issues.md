---
title: Problèmes connus liés à l’approvisionnement d’applications dans Azure AD
description: Découvrez les problèmes connus liés à l’utilisation de l’approvisionnement automatique d’applications dans Azure AD.
author: kenwith
ms.author: kenwith
manager: celestedg
services: active-directory
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: troubleshooting
ms.date: 12/14/2020
ms.reviewer: arvinh
ms.openlocfilehash: 78e9d7355b8c96295456f8aa9cb80b1d3bdd2fab
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97509544"
---
# <a name="known-issues-application-provisioning"></a>Problèmes connus : Approvisionnement d’applications
Problèmes connus à prendre en compte lors de l’utilisation de l’approvisionnement d’applications. Vous pouvez fournir des retours sur le service d’approvisionnement d’applications sur UserVoice ; voir [UserVoice pour l’approvisionnement d’applications Azure AD](https://aka.ms/appprovisioningfeaturerequest). Nous surveillons étroitement le service UserVoice pour améliorer le service. 

> [!NOTE]
> Il ne s’agit pas d’une liste exhaustive des problèmes connus. Si vous découvrez un problème qui ne figure pas dans la liste, partagez vos commentaires en bas de la page.

## <a name="authorization"></a>Autorisation 

**Enregistrement impossible après un test de connexion réussi**

Si vous pouvez tester correctement une connexion, mais que vous ne pouvez pas l’enregistrer, vous avez dépassé la limite de stockage autorisée pour les informations d’identification. Pour plus d’informations, consultez [Problème d’enregistrement des informations d’identification de l’administrateur](./user-provisioning.md).

**Impossible d’enregistrer**

L’URL du locataire, le jeton secret et l’e-mail de notification doivent être renseignés pour être enregistrés. Vous ne pouvez pas simplement fournir une seule de ces données. 

**Impossible de rétablir le mode d’approvisionnement manuel**

Une fois que vous avez configuré l’approvisionnement pour la première fois, vous remarquerez que le mode de configuration est passé de manuel à automatique. Vous ne pouvez pas le repasser sur manuel. Toutefois, vous pouvez désactiver l’approvisionnement par le biais de l’interface utilisateur. La désactivation de l’approvisionnement dans l’interface utilisateur a le même effet la définition de la liste déroulante sur le mode manuel.  


## <a name="attribute-mappings"></a>Mappages d’attributs 

**L’attribut SamAccountName ou userType n’est pas disponible en tant qu’attribut source**

Les attributs SamAccountName et userType ne sont pas disponibles par défaut en tant qu’attributs sources. Développez votre schéma pour ajouter l’attribut. Vous pouvez ajouter les attributs à la liste des attributs sources disponibles en développant votre schéma. Pour plus d’informations, consultez [Attribut source manquant](user-provisioning-sync-attributes-for-mapping.md). 

**Liste déroulante d’attributs sources manquante pour l’extension de schéma**

Les extensions de votre schéma peuvent parfois être absentes dans la liste déroulante d’attributs sources de l’interface utilisateur. Accédez aux paramètres avancés de vos mappages d’attributs et ajoutez manuellement les attributs. Pour plus d’informations, consultez [Personnalisation des mappages d’attributs](customize-application-attributes.md).

**L’attribut null ne peut pas être approvisionné**

Azure AD ne peut actuellement pas approvisionner des attributs null. Si un attribut est null sur l’objet utilisateur, il sera ignoré. 

**Nombre maximal de caractères pour les expressions de mappage d’attributs**

Les expressions de mappage d’attributs peuvent avoir un maximum de 10 000 caractères. 

**Filtres d’étendue non pris en charge**

Les extensions de répertoire appRoleAssignments, userType et accountExpires ne sont pas prises en charge en tant que filtres d’étendue.


## <a name="service-issues"></a>Problèmes liés aux services 

**Scénarios non pris en charge**

- L’approvisionnement de mots de passe n’est pas pris en charge. 
- L’approvisionnement de groupes imbriqués n’est pas pris en charge. 
- L’approvisionnement de locataires B2C n’est pas pris en charge en raison de la taille des locataires.
- Toutes les applications d’approvisionnement ne sont pas disponibles dans tous les clouds. Par exemple, Atlassian n’est pas encore disponible dans le cloud Government. Nous travaillons en collaboration avec les développeurs d’applications pour intégrer leurs applications à tous les clouds.

**Le provisionnement automatique n’est pas disponible sur mon application OIDC**

Si vous créez une inscription d’application, le principal de service correspondant dans les applications d’entreprise n’est pas activé pour le provisionnement automatique d’utilisateurs. Vous devez soit demander l’ajout de l’application à la Galerie, si elle est destinée à être utilisée par plusieurs organisations, soit créer une deuxième application hors Galerie pour le provisionnement. 

**L’intervalle de provisionnement est fixe**

Le [temps](./application-provisioning-when-will-provisioning-finish-specific-user.md#how-long-will-it-take-to-provision-users) entre les cycles de provisionnement n’est actuellement pas configurable. 

**Les modifications ne se déplacent pas de l’application cible vers Azure AD**

Le service d’approvisionnement d’application n’a pas connaissance des modifications apportées aux applications externes. Par conséquent, aucune action n’est effectuée pour la restauration. Le service d’approvisionnement d’application s’appuie sur les modifications apportées à Azure AD. 

**Le cycle d’approvisionnement continue jusqu’à la fin**

Lors de l’approvisionnement de `enabled = off` ou lorsque vous appuyez sur Arrêter, le cycle d’approvisionnement actuel continue à s’exécuter jusqu’à la fin. Le service arrêtera l’exécution des cycles futurs jusqu’à ce que vous réactiviez l’approvisionnement.

**Membre du groupe non approvisionné**

Lorsqu’un groupe est dans l’étendue et qu’un membre est hors de cette étendue, le groupe est approvisionné. L’utilisateur hors de l’étendue n’est pas approvisionné. Si le membre revient dans l’étendue, le service ne détecte pas immédiatement la modification. Le redémarrage de l’approvisionnement permet de résoudre le problème. Nous vous recommandons de redémarrer régulièrement le service pour vous assurer que tous les utilisateurs sont correctement approvisionnés.  

**Le gestionnaire n’est pas approvisionné**

Si un utilisateur et son gestionnaire se trouvent tous deux dans l’étendue d’approvisionnement, le service approvisionne l’utilisateur, puis met à jour le gestionnaire. Toutefois, si au premier jour l’utilisateur se trouve dans l’étendue mais pas le gestionnaire, nous approvisionnons l’utilisateur sans référence au gestionnaire. Une fois le gestionnaire dans l’étendue, la référence au gestionnaire n’est pas mise à jour tant que vous ne redémarrez pas l’approvisionnement pour permettre au service de réévaluer tous les utilisateurs. 

## <a name="next-steps"></a>Étapes suivantes
- [Comment fonctionne le provisionnement](how-provisioning-works.md)
