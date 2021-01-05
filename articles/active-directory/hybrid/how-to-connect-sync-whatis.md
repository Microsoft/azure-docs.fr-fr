---
title: 'Synchronisation d’Azure AD Connect : Comprendre et personnaliser la synchronisation | Microsoft Docs'
description: Explique le fonctionnement et la personnalisation de la synchronisation Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: ee4bf802-045b-4da0-986e-90aba2de58d6
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 11/08/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: cea26cb119f64679807bc6c5eaadb41b341e5d5a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89662378"
---
# <a name="azure-ad-connect-sync-understand-and-customize-synchronization"></a>Synchronisation d’Azure AD Connect : Comprendre et personnaliser la synchronisation
Les services de synchronisation Azure Active Directory Connect (synchronisation Azure AD Connect) sont un composant clé d’Azure AD Connect. Ils se chargent de toutes les opérations liées à la synchronisation des données d’identité entre votre environnement local et Azure AD. Le logiciel Azure AD Connect Sync est le successeur de DirSync, de Microsoft Azure AD Sync et de Forefront Identity Manager, avec le connecteur Azure Active Directory configuré.

Cette rubrique présente le composant **Azure AD Connect Sync** (également appelé **moteur de synchronisation**) et répertorie les liens vers toutes les autres rubriques qui lui sont associées. Pour consulter les liens vers Azure AD Connect, voir [Intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).

Le service de synchronisation se compose de deux composants, le composant local **Azure AD Connect sync** et le composant côté service dans Azure AD, appelé **service de synchronisation Azure AD Connect**.

## <a name="azure-ad-connect-sync-topics"></a>Rubriques de synchronisation Azure AD Connect
| Rubrique | Sujet qu’elle aborde et quand la consulter |
| --- | --- |
| **Principes de base d’Azure AD Connect Sync** | |
| [Présentation de l’architecture](concept-azure-ad-connect-sync-architecture.md) |Pour ceux qui découvrent le moteur de synchronisation et veulent en savoir plus sur l’architecture et les termes utilisés. |
| [Concepts techniques](how-to-connect-sync-technical-concepts.md) |Le sujet de l’architecture est brièvement abordé et les termes utilisés sont expliqués de manière succincte. |
| [Topologies pour Azure AD Connect](plan-connect-topologies.md) |Décrit les différents scénarios et topologies pris en charge par le moteur de synchronisation. |
| **Configuration personnalisée** | |
| [Exécuter à nouveau l’Assistant Installation](how-to-connect-installation-wizard.md) |Explique les options à votre disposition si vous réexécutez l’Assistant Installation d’Azure AD Connect. |
| [Comprendre l’approvisionnement déclaratif](concept-azure-ad-connect-sync-declarative-provisioning.md) |Décrit le modèle de configuration appelé Approvisionnement déclaratif. |
| [Comprendre les expressions d’approvisionnement déclaratif](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md) |Décrit la syntaxe du langage d’expression utilisé dans l’approvisionnement déclaratif. |
| [Présentation de la configuration par défaut](concept-azure-ad-connect-sync-default-configuration.md) |Décrit les règles prêtes à l’emploi et la configuration par défaut. Explique également comment les règles fonctionnent en parallèle pour assurer la réussite des scénarios prêts à l’emploi. |
| [Présentation des utilisateurs et des contacts](concept-azure-ad-connect-sync-user-and-contacts.md) |Suite de la rubrique précédente qui explique comment les configurations associées aux utilisateurs et contacts se complètent, en particulier dans un environnement à forêts multiples. |
| [Comment modifier la configuration par défaut](how-to-connect-sync-change-the-configuration.md) |Vous montre comment modifier une configuration commune pour les flux d’attributs. |
| [Meilleures pratiques pour la modification de la configuration par défaut](how-to-connect-sync-best-practices-changing-default-configuration.md) |Prend en charge les limitations imposées à la configuration fournie par défaut et l’insertion de modifications. |
| [Configurer le filtrage](how-to-connect-sync-configure-filtering.md) |Décrit les différentes options permettant de limiter le nombre d’objets en cours de synchronisation vers Azure AD et explique comment les configurer, étape par étape. |
| **Fonctionnalités et scénarios** | |
| [prévention des suppressions accidentelles](how-to-connect-sync-feature-prevent-accidental-deletes.md) |Décrit la fonctionnalité de *prévention des suppressions accidentelles* et explique comment la configurer. |
| [Scheduler](how-to-connect-sync-feature-scheduler.md) |Décrit le planificateur intégré qui importe, synchronise et exporte les données. |
| [Implémenter la synchronisation de hachage du mot de passe](how-to-connect-password-hash-synchronization.md) |Décrit le fonctionnement de la synchronisation de mot de passe, en indiquant comment l’implémenter, comment l’utiliser et comment résoudre les problèmes associés. |
| [Écriture différée des appareils](how-to-connect-device-writeback.md) |Décrit le fonctionnement de l’écriture différée des appareils dans Azure AD Connect. |
| [Extensions d’annuaire](how-to-connect-sync-feature-directory-extensions.md) |Explique comment étendre le schéma Azure AD avec vos propres attributs personnalisés. |
| [Microsoft 365 PreferredDataLocation](how-to-connect-sync-feature-preferreddatalocation.md) |Décrit comment placer les ressources Microsoft 365 de l’utilisateur dans la même région que l’utilisateur. |
| **Service de synchronisation** | |
| [Fonctionnalités du service de synchronisation Azure AD Connect](how-to-connect-syncservice-features.md) |Décrit le service synchronisation et comment modifier les paramètres de synchronisation dans Azure AD. |
| [Résilience d’attribut en double](how-to-connect-syncservice-duplicate-attribute-resiliency.md) |Décrit comment activer et utiliser la résilience des valeurs d’attribut en double **userPrincipalName** et **proxyAddresses**. |
| **Opérations et interface utilisateur** | |
| [Synchronization Service Manager](how-to-connect-sync-service-manager-ui.md) |Décrit l’interface utilisateur de Synchronization Service Manager, notamment les onglets [Opérations](how-to-connect-sync-service-manager-ui-operations.md), [Connecteurs](how-to-connect-sync-service-manager-ui-connectors.md), [Metaverse Designer](how-to-connect-sync-service-manager-ui-mvdesigner.md) (Concepteur métaverse) et [Metaverse Search](how-to-connect-sync-service-manager-ui-mvsearch.md) (Recherche métaverse). |
| [Considérations et tâches opérationnelles](./how-to-connect-sync-staging-server.md) |Décrit les problèmes liés au fonctionnement, tels que la récupération d’urgence. |
| **Procédure** | |
| [Réinitialiser le compte Azure AD](how-to-connect-azureadaccount.md) |Décrit la réinitialisation des informations d’identification du compte de service utilisé pour se connecter à Azure AD à partir de la synchronisation d’Azure AD Connect. |
| **Plus d’informations et de références** | |
| [Ports](reference-connect-ports.md) |Répertorie les ports que vous devez ouvrir entre le moteur de synchronisation, vos répertoires locaux et Azure AD. |
| [Attributs synchronisés avec Azure Active Directory](reference-connect-sync-attributes-synchronized.md) |Répertorie tous les attributs en cours de synchronisation entre le système AD local et Azure AD. |
| [Référence des fonctions](reference-connect-sync-functions-reference.md) |Répertorie toutes les fonctions disponibles dans l’approvisionnement déclaratif. |

## <a name="additional-resources"></a>Ressources supplémentaires
* [Intégration des identités locales dans Azure Active Directory](whatis-hybrid-identity.md)