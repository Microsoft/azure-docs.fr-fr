---
title: 'Synchronisation d’Azure AD Connect : Concepts techniques | Microsoft Docs'
description: Explique les concepts techniques liés à Azure AD Connect Sync.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 731cfeb3-beaf-4d02-aef4-b02a8f99fd11
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 01/15/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 01e53b30a4c27296e30e031ffb771697afa8e1e9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87019673"
---
# <a name="azure-ad-connect-sync-technical-concepts"></a>Synchronisation d’Azure AD Connect : Concepts techniques
Cet article est un résumé de la rubrique [Présentation de l’architecture](how-to-connect-sync-technical-concepts.md).

Azure AD Connect Sync repose sur une plateforme de synchronisation de méta-annuaire solide.
Les sections suivantes présentent les concepts liés à la synchronisation de méta-annuaire.
S’appuyant sur MIIS (Microsoft Identity Integration Server), ILM (Identity Lifecycle Manager) et FIM (Forefront Identity Manager), les services Azure Active Directory Sync fournissent la plateforme de nouvelle génération pour la connexion aux sources de données. Ils synchronisent les données entre des sources de données et assurent l’approvisionnement et le désapprovisionnement des identités.

![Concepts techniques](./media/how-to-connect-sync-technical-concepts/scenario.png)

Les sections suivantes fournissent plus de détails sur les aspects suivants du service de synchronisation FIM :

* Connecteur
* Flux d’attributs
* Espace de connecteur
* Métaverse
* Approvisionnement

## <a name="connector"></a>Connecteur
Les modules de code utilisés pour communiquer avec un annuaire connecté sont appelés connecteurs (anciennement agents de gestion).

Ils sont installés sur l’ordinateur exécutant Azure AD Connect Sync. Les connecteurs permettent de converser sans agent à l’aide de protocoles système distants, au lieu de reposer sur le déploiement d’agents spécialisés. Cela se traduit par une réduction des risques et de la durée de déploiement, en particulier quand il s’agit de systèmes et d’applications critiques.

Dans l’illustration ci-dessus, le connecteur est synonyme de l’espace de connecteur mais il englobe toutes les communications avec le système externe.

Le connecteur est responsable de toutes les fonctionnalités d’importation et d’exportation vers le système et il évite aux développeurs de devoir comprendre comment se connecter à chaque système en mode natif lors de l’utilisation de l’approvisionnement déclaratif pour personnaliser les transformations de données.

Les importations et exportations ont lieu uniquement quand elles sont planifiées, ce qui offre une isolation supplémentaire par rapport aux modifications qui se produisent dans le système, dans la mesure où les modifications ne se propagent pas automatiquement à la source de données connectée. En outre, les développeurs peuvent également créer leurs propres connecteurs pour se connecter à pratiquement n'importe quelle source de données.

## <a name="attribute-flow"></a>Flux d’attributs
Le métaverse est l’affichage consolidé de toutes les identités jointes des espaces de connecteur voisins. Dans la figure ci-dessus, le flux des attributs est représenté par des lignes comportant des flèches pour les flux entrant et sortant. Le flux des attributs est le processus de copie ou de transformation de données d'un système vers un autre et vers tous les flux d’attributs (entrants ou sortants).

Le flux d’attributs se produit entre l’espace de connecteur et le métaverse de manière bidirectionnelle quand l’exécution d’opérations de synchronisation (complète ou delta) est planifiée.

Le flux d’attributs se produit uniquement quand ces synchronisations sont exécutées. Les flux d’attributs sont définis dans des règles de synchronisation. Ces règles peuvent être entrantes (ISR dans l’image ci-dessus) ou sortantes (OSR dans l’image ci-dessus).

## <a name="connected-system"></a>Système connecté
Système connecté (également appelé annuaire connecté) fait référence au système distant auquel Azure AD Connect Sync s'est connecté et vers lequel et à partir duquel il lit et écrit des données d'identité.

## <a name="connector-space"></a>Espace de connecteur
Chaque source de données connectée est représentée comme un sous-ensemble filtré des objets et des attributs dans l’espace de connecteur.
Cela permet au service de synchronisation de s’exécuter localement sans qu’il soit nécessaire de contacter le système distant lors de la synchronisation des objets et cela limite l’interaction aux importations et exportations.

Quand la source de données et le connecteur peuvent fournir une liste de modifications (une importation delta), l’efficacité opérationnelle augmente considérablement car seules les modifications apportées depuis le dernier cycle d’interrogation sont échangées. L’espace de connecteur isole la source de données connectée des modifications qui se propagent automatiquement en exigeant que le connecteur planifie les importations et les exportations. Cette assurance supplémentaire vous procure une tranquillité d’esprit lors des tests, de l’examen ou de la confirmation de la mise à jour suivante.

## <a name="metaverse"></a>Métaverse
Le métaverse est l’affichage consolidé de toutes les identités jointes des espaces de connecteur voisins.

À mesure que des identités sont liées et que l’autorité est attribuée pour différents attributs via des mappages de flux d’importation, l’objet de métaverse central commence à regrouper les informations provenant de plusieurs systèmes. À partir de ce flux d’attributs d’objets, des mappages transmettent des informations aux systèmes sortants.

Des objets sont créés quand un système faisant autorité les projette dans le métaverse. Dès que toutes les connexions sont supprimées, l’objet de métaverse est supprimé.

Impossible de modifier directement les objets de métaverse. Toutes les données de l'objet doivent être fournies via le flux des attributs. Le métaverse conserve les connecteurs permanents avec chaque espace de connecteur. Ces connecteurs ne nécessitent pas de réévaluation pour chaque exécution de la synchronisation. Cela signifie qu'Azure AD Connect Sync n'a pas à localiser à chaque fois l'objet distant correspondant. On n'a donc pas besoin d'agents onéreux pour empêcher des modifications des attributs qui seraient normalement responsables de la mise en corrélation des objets.

Lors de la découverte de nouvelles sources de données pouvant contenir des objets à gérer, Azure AD Connect Sync utilise un processus appelé règle de jointure pour évaluer les candidats potentiels avec lesquels établir un lien.
Une fois le lien établi, cette évaluation ne se reproduit pas et le flux d’attributs normal peut se produire entre la source de données connectée et le métaverse.

## <a name="provisioning"></a>Approvisionnement
Quand une source faisant autorité projette un nouvel objet dans le métaverse, un nouvel objet d’espace de connecteur peut être créé dans un autre connecteur représentant une source de données connectée en aval.

Cela établit intrinsèquement un lien, et le flux d’attributs peut se produire de manière bidirectionnelle.

Chaque fois qu’une règle détermine qu’un nouvel objet d’espace de connecteur doit être créé, on emploie le terme d’« approvisionnement ». Toutefois, étant donné que cette opération n’a lieu que dans l’espace de connecteur, elle n’est reportée dans la source de données connectée qu’une fois qu’une exportation est effectuée.

## <a name="additional-resources"></a>Ressources supplémentaires
* [Synchronisation Azure AD Connect : personnaliser les options de synchronisation](how-to-connect-sync-whatis.md)
* [Intégration des identités locales dans Azure Active Directory](whatis-hybrid-identity.md)

<!--Image references-->
[1]: ./media/active-directory-aadsync-technical-concepts/ic750598.png
