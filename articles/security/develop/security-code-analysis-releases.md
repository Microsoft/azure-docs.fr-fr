---
title: Versions Microsoft Security Code Analysis
description: Cet article décrit les prochaines versions de l’extension Microsoft Security Code Analysis
author: sukhans
manager: sukhans
ms.author: terrylan
ms.date: 04/24/2020
ms.topic: article
ms.service: security
services: azure
ms.assetid: 521180dc-2cc9-43f1-ae87-2701de7ca6b8
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: na
ms.openlocfilehash: 33ce2a496caa52609d8bdf8c92e29064ca4ae349
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85362038"
---
# <a name="microsoft-security-code-analysis-releases-and-roadmap"></a>Versions Microsoft Security Code Analysis et feuille de route

En partenariat avec Developer Support, l’équipe d’analyse Microsoft Security Code Analysis est fière d’annoncer les améliorations récentes et à venir de notre extension MSCA.


## <a name="credential-scanner-v20-released-in-april-2020"></a>Credential Scanner v2.0 : Publié en avril 2020

### <a name="innovations--improvements"></a>Innovations et améliorations

- **Moteur principal**

   - Mise à niveau des performances moyennes de 25 % avec des délais d’exécution quasi linéaires
   - Recherche et classement en fonction du contexte et des preuves pour une plus grande précision
   - Améliorations apportées aux détections de mot de passe générales et à la logique de correspondance pour les espaces réservés évidents (par exemple, fakePassword)

- **Couverture** : prise en charge de plus de 25 types de secrets, y compris les éléments suivants les plus demandés :

   - Phrase secrète du certificat du compte Service Fabric
   - Secret/clé API du client
   - En-tête d'autorisation HTTP
   - Clé d’accès à la clé secrète du client Amazon S3
   - Jeton d’accès au client Azure Active Directory
   - Clé principale/API Azure Function
   - Clé d’accès Power BI
   - Modèle de mot de passe Azure Resource Manager

- **Sorties**

   - Prise en charge des formats de fichier de sortie SARIF 2.1 et CSV

## <a name="binskim-v160-released-in-april-2020"></a>BinSkim v1.6.0 : Publié en avril 2020

### <a name="improvements"></a>Améliorations

- FONCTIONNALITÉ : Mise à jour vers la version finale SARIF v2 (version 2.1.16). Cette mise à jour permet la mise en cache des résultats lors du passage des hachages sur la ligne de commande, ce qui améliore considérablement les performances lors de l’analyse récursive des répertoires avec plusieurs copies des cibles d’analyse.
- RÉSOLUTION DE BOGUE : Correction d’une faute de frappe dans la sortie BA2021.DoNotMarkWritableSectionsAsExecutable.
- PERFORMANCES : Éliminez le chargement PDB pour tous les assemblys managés qui ne sont pas en mode mixte, y compris les binaires de la bibliothèque IL (par anticipation).
- CORRECTIF DE FAUX NÉGATIF : Vérifier qu’un fichier PDB placé avec un binaire correspond réellement au binaire en cours d’analyse
- FONCTIONNALITÉ : Fournit un argument --local-symbol-directories pour spécifier des emplacements de recherche PDB supplémentaires (locaux, serveur sans symbole)
- CORRECTIF DE FAUX POSITIF : Ignorez l’analyse pilotée par PDB pour le fichier exécutable natif .NET Core généré (code non contrôlable par l’utilisateur).

## <a name="whats-next-in-q3-cy20"></a>Quelles sont les prochaines étapes pour le troisième trimestre 2020 ?

- Outil d’analyse de la sécurité Java
- Outil d’analyse de la sécurité Python
- ES Lint remplacera TS Lint pour TypeScript et JavaScript
- Outil d’analyse des modèles Resource Manager

## <a name="tool-deprecation-notification"></a>Notification de dépréciation de l’outil

### <a name="microsoft-security-risk-detection-msrd-is-deprecated-on-june-26-2020"></a>Microsoft Security Risk Detection (MSRD) est dépréciée le 26 juin 2020.

Le service de test aléatoire MSRD déprécié sera remplacé par une plateforme de test aléatoire de développeur auto-hébergée open source pour Azure. Cette plateforme est actuellement en cours de développement et testée en partenariat avec de nombreuses équipes de produits principales de Microsoft. Cette plateforme de test aléatoire intègre des assainisseurs et permet d’effectuer des tests aléatoires d’apprentissage intégrés dans des pipelines CI/CD qui évoluent avec le temps avec des projets logiciels. La version open source de cette plateforme est prévue pour le deuxième semestre 2020.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir des instructions sur l'intégration et l'installation de Microsoft Security Code Analysis, consultez notre [Guide d'intégration et d'installation](security-code-analysis-onboard.md).

Si vous avez d’autres questions sur l’extension et les outils proposés, voir notre [page FAS](security-code-analysis-faq.md).
