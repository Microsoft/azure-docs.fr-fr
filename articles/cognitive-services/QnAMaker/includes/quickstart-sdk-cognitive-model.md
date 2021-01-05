---
ms.openlocfilehash: c87b9dc22ecd937abb27417aeddc1e30c0d724e7
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "85114524"
---

## <a name="using-this-example-knowledge-base"></a>Utilisation de cet exemple de base de connaissances

Dans ce guide de démarrage rapide, la base de connaissances commence par deux paires de questions et réponses conversationnelles, le but étant de simplifier l’exemple et d’obtenir des ID hautement prévisibles à utiliser dans la méthode de mise à jour, en associant les invites de suivi des questions avec de nouvelles paires. Tout cela a été planifié et implémenté dans un ordre spécifique pour les besoins de ce guide de démarrage rapide.

Si vous prévoyez de développer votre base de connaissances au fil du temps avec des invites de suivi qui dépendent de paires de questions et réponses existantes, vous pouvez choisir les méthodes suivantes :
* Pour des bases de connaissances volumineuses, gérez la base de connaissances dans un éditeur de texte ou un outil TSV qui prend en charge l’automatisation, puis remplacez entièrement la base de connaissances en une seule fois par le biais d’une mise à jour.
* Pour des bases de connaissances de petite taille, gérez intégralement les invites de suivi dans le portail QnA Maker.

Informations sur les paires de questions/réponses utilisées dans ce guide de démarrage rapide :
* Types de paires de questions/réponses : il existe deux types de paires de questions/réponses dans cette base de connaissances après la mise à jour : les informations d’échange (chitchat) et celles propres au domaine. C’est généralement le cas si votre base de connaissances est liée à une application de conversation telle qu’un chatbot.
* Les réponses de la base de connaissances peuvent être filtrées par métadonnées ou utiliser des invites de suivi, mais ce guide de démarrage rapide ne le montre pas. Consultez ces exemples generateAnswer indépendants du langage [ici](../Quickstarts/get-answer-from-knowledge-base-using-url-tool.md).
* Le texte de réponse est au format Markdown. Il peut contenir une [grande variété de contenu Markdown](../reference-markdown-format.md) comme des images (images disponibles publiquement sur Internet), des liens (vers des URL accessibles publiquement) et des éléments à puce, mais ce guide de démarrage rapide ne reflète pas cette variété.
