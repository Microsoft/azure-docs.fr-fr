---
title: 'Synchronisation d’Azure AD Connect : Changement de la configuration par défaut | Microsoft Docs'
description: Décrit les meilleures pratiques pour la modification de la configuration par défaut d’Azure AD Connect Sync.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 7638a031-1635-4942-94c3-fce8f09eed5e
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/29/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 70e91ff8fa3666a2dfc5aaad07be7927852b08bd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85357696"
---
# <a name="azure-ad-connect-sync-best-practices-for-changing-the-default-configuration"></a>Synchronisation d’Azure AD Connect : Meilleures pratiques pour la modification de la configuration par défaut
L’objectif de cette rubrique est de décrire les modifications prises en charge et celles qui ne sont pas prises en charge pour Azure AD Connect Sync.

La configuration créée par Azure AD Connect fonctionne « telle quelle » pour la plupart des environnements qui synchronisent Active Directory local avec Azure AD. Toutefois, dans certains cas, il est nécessaire d’appliquer certaines modifications à une configuration pour répondre à une spécification ou un besoin particulier.

## <a name="changes-to-the-service-account"></a>Modifications apportées au compte de service
Azure AD Connect Sync s’exécute sous un compte de service créé par l’Assistant Installation. Ce compte de service conserve les clés de chiffrement dans la base de données utilisée par la synchronisation. Il est créé avec un mot de passe de 127 caractères de long, défini pour ne pas expirer.

* La modification ou la réinitialisation du mot de passe du compte de service **n’est pas prise en charge** . Sa modification ou sa réinitialisation supprime les clés de chiffrement et le service ne peut pas accéder à la base de données ni démarrer.

## <a name="changes-to-the-scheduler"></a>Modifications apportées au planificateur
À partir de la version 1.1 (février 2016), vous pouvez configurer le [planificateur](how-to-connect-sync-feature-scheduler.md) sur un cycle de synchronisation autre que le cycle par défaut de 30 minutes.

## <a name="changes-to-synchronization-rules"></a>Modifications apportées aux règles de synchronisation
L'Assistant d’installation fournit une configuration qui est censée fonctionner pour les scénarios les plus courants. Au cas où vous devez apporter des modifications à la configuration, vous devez suivre ces règles pour toujours avoir une configuration prise en charge.

> [!WARNING]
> Si vous apportez des modifications aux règles de synchronisation par défaut, ces modifications seront remplacées lors de la prochaine mise à jour d’Azure AD Connect, entraînant des résultats de synchronisation inattendus et probablement indésirables.

* Vous pouvez [modifier le flux d’attributs](how-to-connect-sync-change-the-configuration.md#other-common-attribute-flow-changes) si le flux des attributs directs par défaut ne convient pas à votre organisation.
* Si vous ne souhaitez [pas transférer un attribut](how-to-connect-sync-change-the-configuration.md#do-not-flow-an-attribute) et voulez supprimer des valeurs d’attribut d’Azure AD existantes, vous devez créer une règle pour ce scénario.
* [Désactiver une règle de synchronisation indésirable](#disable-an-unwanted-sync-rule) plutôt que la supprimer. Une règle supprimée est recréée pendant la mise à niveau.
* Pour [effectuer des modifications sur une règle out-of-box](#change-an-out-of-box-rule), vous devez faire une copie de la règle d’origine, puis la désactiver. L’éditeur de règles de synchronisation affiche une invite et vous propose de l’aide.
* Exportez vos règles de synchronisation personnalisées à l’aide de l’éditeur de règles de synchronisation. L’éditeur vous donne un script PowerShell que vous pouvez utiliser pour facilement les recréer dans le cadre d’une récupération d’urgence.

> [!WARNING]
> Les règles de synchronisation out-of-box ont une empreinte numérique. Si vous apportez une modification à ces règles, l’empreinte numérique ne correspondra plus. Vous risquez de rencontrer des problèmes lorsque vous tenterez d’appliquer une nouvelle version d’Azure AD Connect. Procédez à des modifications uniquement de la façon décrite dans cet article.

### <a name="disable-an-unwanted-sync-rule"></a>Désactiver une règle de synchronisation indésirable
Ne supprimez pas une règle de synchronisation out-of-box. Elle est recréée au cours de la mise à niveau suivante.

Dans certains cas, l'Assistant d'installation a produit une configuration qui ne fonctionne pas pour votre topologie. Par exemple, si vous disposez d'une topologie de forêt compte-ressource, mais que vous avez étendu le schéma dans la forêt de comptes avec le schéma Exchange, alors les règles pour Exchange sont créées pour la forêt de comptes, ainsi que pour la forêt de ressources. Dans ce cas, vous devez désactiver la règle de synchronisation pour Exchange.

![Règle de synchronisation désactivée](./media/how-to-connect-sync-best-practices-changing-default-configuration/exchangedisabledrule.png)

Dans l'image ci-dessus, l'Assistant d'installation a trouvé un ancien schéma Exchange 2003 dans la forêt de comptes. Cette extension de schéma a été ajoutée avant l’introduction de la forêt de ressources dans l’environnement de Fabrikam. Pour garantir qu’aucun attribut de l’ancienne implémentation Exchange n’est synchronisé, la règle de synchronisation devrait être désactivée comme indiqué.

### <a name="change-an-out-of-box-rule"></a>effectuer des modifications sur une règle out-of-box
La seule fois où vous devez effectuer des modifications sur une règle out-of-box est lorsque vous devez modifier la règle de jointure. Si vous devez modifier un flux d’attributs, vous devez créer une règle de synchronisation avec une priorité plus élevée que les règles out-of-box. La seule règle que vous aurez vraiment à cloner est la règle **Reçu depuis AD - jointure utilisateur**. Vous pouvez remplacer toutes les autres règles avec une règle de priorité plus élevée.

Si vous devez effectuer des modifications d’une règle out-of-box, vous devriez en faire une copie, puis désactiver la règle d’origine. Modifiez ensuite la règle clonée. L’éditeur de règles de synchronisation vous accompagne dans ces étapes. Lorsque vous ouvrez une règle out-of-box, cette boîte de dialogue s'affiche :  
![Règle d'avertissement out-of-the box](./media/how-to-connect-sync-best-practices-changing-default-configuration/warningoutofboxrule.png)

Sélectionnez **Oui** pour créer une copie de la règle. La règle clonée est ensuite ouverte.  
![Règle clonée](./media/how-to-connect-sync-best-practices-changing-default-configuration/clonedrule.png)

Sur cette règle clonée, effectuez les modifications nécessaires de la portée, de la jointure et des transformations.

## <a name="next-steps"></a>Étapes suivantes
**Rubriques de présentation**

* [Synchronisation Azure AD Connect : Comprendre et personnaliser la synchronisation](how-to-connect-sync-whatis.md)
* [Intégration des identités locales dans Azure Active Directory](whatis-hybrid-identity.md)
