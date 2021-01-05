---
title: Résoudre les problèmes d’un attribut ne synchronisant pas dans Azure AD Connect | Microsoft Docs
description: Cet article explique comment résoudre des problèmes de synchronisation d’attribut à l’aide de la tâche de dépannage.
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
ms.date: 01/31/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: a6df1347eab57a6971fe2e39c0a55869c8f23939
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91317485"
---
# <a name="troubleshoot-an-attribute-not-synchronizing-in-azure-ad-connect"></a>Résoudre les problèmes d’un attribut ne synchronisant pas dans Azure AD Connect

## <a name="recommended-steps"></a>**Étapes recommandées**

Avant d’examiner les problèmes de synchronisation des attributs, essayons de comprendre le processus de synchronisation d’**Azure AD Connect** :

  ![Processus de synchronisation Azure AD Connect](media/tshoot-connect-attribute-not-syncing/tshoot-connect-attribute-not-syncing/syncingprocess.png)

### <a name="terminology"></a>**Terminologie**

* **CS :** Connector Space (table de base de données).
* **MV :** métaverse (table de base de données).
* **AD :** Active Directory
* **AAD :** Azure Active Directory

### <a name="synchronization-steps"></a>**Étapes de la synchronisation**

* Importer à partir d’AD : les objets Active Directory sont placés dans AD CS.

* Importer à partir d’AAD : les objets Azure Active Directory sont placés dans AAD CS.

* Synchronisation : les **règles de synchronisation entrante** et les **règles de synchronisation sortante** sont exécutées dans l’ordre de priorité des nombres du plus petit au plus grand. Pour afficher les règles de synchronisation, vous pouvez accéder à **Synchronization Rules Editor** (Éditeur des règles de synchronisation) à partir des applications de bureau. Les **règles de synchronisation entrante** placent les données de CS dans MV. Les **règles de synchronisation sortante** déplacent les données de MV dans CS.

* Exporter vers AD : Une fois la synchronisation effectuée, les objets sont exportés depuis AD CS vers **Active Directory**.

* Exporter vers AAD : Une fois la synchronisation effectuée, les objets sont exportés depuis AAD CS vers **Azure Active Directory**.

### <a name="step-by-step-investigation"></a>**Examen pas à pas**

* Nous allons commencer notre recherche à partir du **métaverse** et examiner le mappage d’attribut de la source à la cible.

* Lancez **Synchronization Service Manager** à partir des applications de bureau, comme indiqué ci-dessous :

  ![Lancer Synchronization Service Manager](media/tshoot-connect-attribute-not-syncing/tshoot-connect-attribute-not-syncing/startmenu.png)

* Dans **Synchronization Service Manager**, sélectionnez **Metaverse Search** (Recherche de métaverse), **Scope by Object Type** (étendue par type d’objet), choisissez l’objet à l’aide d’un attribut, puis cliquez sur le bouton **Recherche**.

  ![Recherche de métaverse](media/tshoot-connect-attribute-not-syncing/tshoot-connect-attribute-not-syncing/mvsearch.png)

* Double-cliquez sur l’objet trouvé dans la recherche de **métaverse** pour afficher tous ses attributs. Vous pouvez cliquer sur l’onglet **Connectors** (Connecteurs) pour examiner un objet correspondant dans tous les **Connector Spaces** (Espaces connecteur).

  ![Connecteurs d’objet de métaverse](media/tshoot-connect-attribute-not-syncing/tshoot-connect-attribute-not-syncing/mvattributes.png)

* Double-cliquez sur le **connecteur Active Directory** pour afficher les attributs du **Connector Space** (Espace connecteur). Cliquez sur le bouton **Aperçu**, puis, dans la boîte de dialogue suivante, sur le bouton **Générer l’aperçu**.

  ![Capture d’écran montrant l’écran Propriétés de l’objet d’espace connecteur avec le bouton Aperçu en évidence.](media/tshoot-connect-attribute-not-syncing/tshoot-connect-attribute-not-syncing/csattributes.png)

* Cliquez à présent sur **Import Attribute Flow** (Importer le flux d’attributs). Cela a pour effet de présenter le flux d’attributs de l’**espace connecteur Active Directory** au **métaverse**. La colonne **Sync Rule** (Règle de synchronisation) indique la **règle de synchronisation** qui a contribué à cet attribut. La colonne **Data Source** (Source de données) indique les attributs de l’**espace connecteur**. La colonne **Metaverse Attribute** (Attribut du métaverse) indique les attributs dans le **métaverse**. Vous pouvez rechercher l’attribut qui ne se synchronise pas ici. Si vous ne trouvez pas l’attribut ici, cela signifie qu’il n’est pas mappé et que vous devez créer une **règle de synchronisation** personnalisée pour mapper l’attribut.

  ![Attributs d’espace connecteur](media/tshoot-connect-attribute-not-syncing/tshoot-connect-attribute-not-syncing/cstomvattributeflow.png)

* Cliquez sur **Export Attribute Flow** (Exporter le flux d’attributs) dans le volet gauche pour présenter le flux d’attributs de **métaverse** à l’**espace connecteur Active Directory** en utilisant des **Règles de synchronisation sortante**.

  ![Capture d’écran montrant le flux d’attributs du métaverse à l’espace connecteur Active Directory utilisant des règles de synchronisation sortante.](media/tshoot-connect-attribute-not-syncing/tshoot-connect-attribute-not-syncing/mvtocsattributeflow.png)

* De même, vous pouvez afficher l’objet **Azure Active Directory Connector Space** (Espace connecteur Azure Active Directory) et générer l’**aperçu** pour présenter le flux d’attributs du **métaverse** à l’**espace connecteur** et vice versa, de façon à pouvoir examiner pourquoi un attribut n’est pas synchronisé.

## <a name="recommended-documents"></a>**Documents recommandés**
* [Synchronisation Azure AD Connect : concepts techniques](./how-to-connect-sync-technical-concepts.md)
* [Synchronisation Azure AD Connect : comprendre l’architecture](./concept-azure-ad-connect-sync-architecture.md)
* [Synchronisation Azure AD Connect : présentation du provisionnement déclaratif](./concept-azure-ad-connect-sync-declarative-provisioning.md)
* [Synchronisation Azure AD Connect : comprendre les expressions de provisionnement déclaratif](./concept-azure-ad-connect-sync-declarative-provisioning-expressions.md)
* [Synchronisation Azure AD Connect : comprendre la configuration par défaut](./concept-azure-ad-connect-sync-default-configuration.md)
* [Synchronisation Azure AD Connect : Comprendre les utilisateurs, les groupes et les contacts](./concept-azure-ad-connect-sync-user-and-contacts.md)
* [Synchronisation Azure AD Connect : attributs de clichés instantanés](./how-to-connect-syncservice-shadow-attributes.md)

## <a name="next-steps"></a>Étapes suivantes

- [Synchronisation Azure AD Connect](how-to-connect-sync-whatis.md).
- [Qu’est-ce que l’identité hybride ?](whatis-hybrid-identity.md).