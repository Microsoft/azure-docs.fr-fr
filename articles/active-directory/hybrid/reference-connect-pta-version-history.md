---
title: 'Authentification directe Azure AD : Historique de publication des versions | Microsoft Docs'
description: Cet article répertorie toutes les mises en production de l’agent d’authentification directe Azure AD
services: active-directory
author: billmath
manager: daveba
ms.assetid: ef2797d7-d440-4a9a-a648-db32ad137494
ms.service: active-directory
ms.topic: reference
ms.workload: identity
ms.date: 04/14/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 854931273ffebd653615a44dbfa9ce1b846655c2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91333424"
---
# <a name="azure-ad-pass-through-authentication-agent-version-release-history"></a>Agent d’authentification directe Azure AD : Historique de publication des versions 
 
Les agents installés localement et qui activent l’authentification directe sont régulièrement mis à jour pour fournir de nouvelles fonctionnalités. Cet article répertorie les versions et les fonctionnalités ajoutées lors de l’introduction de nouvelles fonctionnalités. Les agents d’authentification directe sont mis à jour automatiquement lorsqu’une nouvelle version est mise en production. 

Voici les rubriques connexes : 

- [Connexion de l’utilisateur avec l’authentification directe Azure AD](how-to-connect-pta.md) 
- [Installation de l’agent d’authentification directe Azure AD](how-to-connect-pta-quick-start.md) 

## <a name="1517420"></a>1.5.1742.0
### <a name="release-status"></a>Statut de la version : 
09/04/2020 : publiée pour téléchargement

### <a name="new-features-and-improvements"></a>Améliorations et nouvelles fonctionnalités

- Ajout de la prise en charge du ciblage des environnements cloud après l’installation. Le bundle peut être destiné à un environnement cloud donné.



## <a name="1510070"></a>1.5.1007.0 
### <a name="release-status"></a>État de la version 
22/1/2019 : publiée pour téléchargement  
### <a name="new-features-and-improvements"></a>Améliorations et nouvelles fonctionnalités 
- Support ajouté pour des canaux fiables Service Bus pour ajouter une autre couche de résilience des connexions pour les connexions sortantes 
- Appliquer TLS 1.2 lors de l’inscription de l’agent 

## <a name="156430"></a>1.5.643.0 
### <a name="release-status"></a>État de la version 
10/4/2018 : publiée pour téléchargement  
### <a name="new-features-and-improvements"></a>Améliorations et nouvelles fonctionnalités 
- Support des connexions socket Web 
- Définissez TLS 1.2 comme protocole par défaut pour l’agent 
 
## <a name="154050"></a>1.5.405.0 
### <a name="release-status"></a>État de la version 
31/1/2018 : publiée pour téléchargement  
### <a name="fixed-issues"></a>Problèmes résolus 
- Correction d’un bogue entraînant des fuites de mémoire dans l’agent. 
- Mise à jour de la version d’Azure Service Bus, comprenant un correctif des problèmes de délais d’expiration du connecteur. 
### <a name="new-features-and-improvements"></a>Améliorations et nouvelles fonctionnalités 
- Support ajouté pour des connexions WebSocket entre l’agent et les services de Azure AD pour améliorer la résilience des connexions

## <a name="154020"></a>1.5.402.0 
### <a name="release-status"></a>État de la version 
25/11/2017 : publiée pour téléchargement  
### <a name="fixed-issues"></a>Problèmes résolus 
- Bogues corrigés liés au cache DNS pour les scénarios de proxy par défaut 
 
## <a name="153890"></a>1.5.389.0 
### <a name="release-status"></a>État de la version 
17/10/2017 : publiée pour téléchargement  
### <a name="new-features-and-improvements"></a>Améliorations et nouvelles fonctionnalités 
- Fonctionnalité de cache DNS ajoutée pour des connexions sortantes afin d’ajouter la résilience contre les défaillances DNS 
 
## <a name="152610"></a>1.5.261.0 
### <a name="release-status"></a>État de la version 
31/8/2017 : publiée pour téléchargement  
### <a name="new-features-and-improvements"></a>Améliorations et nouvelles fonctionnalités 
- Version GA de l’agent d’authentification directe Azure AD 

## <a name="next-steps"></a>Étapes suivantes

- [Connexion de l’utilisateur avec l’authentification directe Azure Active Directory](how-to-connect-pta.md)
