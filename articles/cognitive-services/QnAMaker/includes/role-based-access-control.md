---
title: Fichier include
description: Fichier include
ms.topic: include
ms.custom: include file
ms.date: 03/11/2020
ms.openlocfilehash: b34d3ed2481600cde7ffb3275d635b8a57926bd6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83665920"
---
Les rôles suivants sont fournis pour la collaboration :

|Role|Fonctionnalités|Accès à l’interface de programmation d’applications|Autorisations des API|
|--|--|--|--|
|Propriétaire|Tous|Clé d’authentification|Tous|
|Contributeur|Tout sauf la possibilité d’ajouter de nouveaux membres aux rôles|Clé d’authentification|Tout sauf la possibilité d’ajouter de nouveaux membres aux rôles|
|Lecture QnA Maker<br>(lecture)|Export/Téléchargement<br>Test|Jeton du porteur|1. Téléchargement de l’API KB<br>2. Classement des API KB pour l’API utilisateur<br>3. Obtenir des détails sur la base de connaissances<br>4. Téléchargement des modifications<br>Génération de la réponse |
|Éditeur QnA Maker<br>(lecture/écriture)|Export/Téléchargement<br>Test<br>Mise à jour KB<br>Exportation KB<br>Importation KB<br>Remplacement KB<br>Créer la base de connaissances|Jeton du porteur|1. Création de l’API KB<br>2. Mise à jour de l’API KB<br>3. Remplacement de l’API KB<br>4. Remplacement des modifications<br>5. « API Train » [dans le nouveau modèle de service v5]|
|Utilisateur du service cognitif<br>(lecture/écriture/publication)|Tous|Jeton du porteur|Tous|