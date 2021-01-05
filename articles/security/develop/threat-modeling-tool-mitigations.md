---
title: Atténuations - Outil Microsoft de modélisation des menaces - Azure | Documents Microsoft
description: Page des atténuations de l’outil Microsoft de modélisation des menaces, mettant en évidence des solutions possibles pour les menaces générées les plus exposées.
services: security
documentationcenter: na
author: jegeib
manager: jegeib
editor: jegeib
ms.assetid: na
ms.service: security
ms.subservice: security-develop
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/17/2017
ms.author: rodsan
ms.openlocfilehash: 748d10b994080b667885e5d0d5f4d688269e86ab
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "68728044"
---
# <a name="microsoft-threat-modeling-tool-mitigations"></a>Atténuations de l’outil Microsoft de modélisation des menaces

L’outil de modélisation des menaces est un élément essentiel de Microsoft Security Development Lifecycle (SDL). Il permet aux architectes logiciels d’identifier et de corriger les problèmes de sécurité potentiels au plus tôt, lorsqu’ils sont relativement simples et peu coûteux à résoudre. Par conséquent, il réduit considérablement le coût total de développement. En outre, nous avons conçu cet outil en pensant aux personnes qui ne sont pas expertes en sécurité et nous avons facilité la modélisation des menaces pour tous les développeurs en fournissant des conseils éclairés sur la création et l’analyse des modèles de menace.

Visitez **[l’outil de modélisation des menaces](threat-modeling-tool.md)** pour démarrer dès aujourd'hui !

## <a name="mitigation-categories"></a>Catégories d’atténuation

Les corrections de l’outil de modélisation des menaces sont classifiées en fonction de l’infrastructure de sécurité des applications web, comportant les éléments suivants :

| Category | Description |
| -------- | ----------- |
| **[Audit et journalisation](threat-modeling-tool-auditing-and-logging.md)** | Qui a fait quoi et quand ? L’audit et la journalisation font référence à la manière dont votre application enregistre les événements liés à la sécurité |
| **[Authentification](threat-modeling-tool-authentication.md)** | qui êtes-vous ? L’authentification désigne le processus dans lequel une entité prouve l’identité d’une autre entité, généralement par le biais d’informations d’identification, comme le nom d’utilisateur et le mot de passe |
| **[Autorisation](threat-modeling-tool-authorization.md)** | que pouvez-vous faire ? L’autorisation désigne la manière dont votre application fournit des contrôles d’accès pour les ressources et les opérations |
| **[Sécurité des communications](threat-modeling-tool-communication-security.md)** | À qui parlez-vous ? La sécurité des communications garantit que toutes les communications effectuées sont aussi sécurisées que possible |
| **[Gestion des configurations](threat-modeling-tool-configuration-management.md)** | Pour qui votre application s’exécute-t-elle ? À quelles bases de données se connecte-t-elle ? Comment votre application est-elle administrée ? Comment ces paramètres sont-ils sécurisés ? La gestion de la configuration fait référence à la façon dont votre application gère ces problèmes opérationnels |
| **[Cryptographie](threat-modeling-tool-cryptography.md)** | Comment conservez-vous les secrets (confidentialité) ? Comment protégez-vous vos données ou vos bibliothèques de la falsification (intégrité) ? Comment obtenez-vous des valeurs aléatoires solides d’un point de vue cryptographique ? La cryptographie fait référence à la façon dont votre application applique les principes de confidentialité et d’intégrité |
| **[Gestion des exceptions](threat-modeling-tool-exception-management.md)** | En cas d’échec d’un appel de méthode dans votre application, que fait votre application ? Dans quelle mesure révélez-vous vos informations ? Renvoyez-vous des messages d’erreur conviviaux aux utilisateurs finaux ? Transmettez-vous des informations d’exception utiles à l’appelant ? Votre application échoue-t-elle de manière appropriée ? |
| **[Validation des entrées](threat-modeling-tool-input-validation.md)** | Comment savez-vous que l’entrée reçue par votre application est valide et sécurisée ? La validation des entrées fait référence à la manière dont votre application filtre, nettoie ou refuse les entrées avant tout traitement supplémentaire. Pensez à limiter les entrées par le biais des points d’entrée et à encoder les sorties par le biais des points de sortie. Faites-vous confiance aux données provenant de sources telles que les bases de données et les partages de fichiers ? |
| **[Données sensibles](threat-modeling-tool-sensitive-data.md)** | Comment votre application gère-t-elle les données sensibles ? Les données sensibles font référence à la manière dont votre application gère toutes les données devant être protégées en mémoire, sur le réseau, ou dans les magasins persistants |
| **[Gestion des sessions](threat-modeling-tool-session-management.md)** | Comment votre application gère-t-elle et protège-t-elle les sessions utilisateur ? Une session fait référence à une série d’interactions associées entre un utilisateur et votre application web |

Cela vous permet d’identifier :

* Où les erreurs les plus courantes se produisent
* Où les améliorations les plus exploitables sont effectuées

Par conséquent, vous utilisez ces catégories pour cibler et hiérarchiser votre travail de sécurité : si vous connaissez les problèmes de sécurité les plus courants dans les catégories de validation, d’authentification et d’autorisation des entrées, vous pouvez démarrer par là. Pour plus d’informations, consultez **[ce lien de brevet](https://www.google.com/patents/US7818788)**

## <a name="next-steps"></a>Étapes suivantes

Visitez **[Menaces pour l’outil de modélisation des menaces](threat-modeling-tool-threats.md)** pour en savoir plus sur les catégories de menaces que l’outil utilise pour générer des menaces de conception possible.