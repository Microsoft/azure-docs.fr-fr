---
title: Assistants vocaux sur Windows - Directives de conception
titleSuffix: Azure Cognitive Services
description: Bonnes pratiques pour la conception d’une expérience d’agent vocal.
services: cognitive-services
author: cfogg6
manager: spencer.king
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/1/2020
ms.author: adamwa
ms.openlocfilehash: 76a29af80891142fcf7f56f93f5c7acc5c58ab04
ms.sourcegitcommit: 419c8c8061c0ff6dc12c66ad6eda1b266d2f40bd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/18/2020
ms.locfileid: "92165143"
---
# <a name="design-assistant-experiences-for-windows-10"></a>Concevoir des expériences d’assistant pour Windows 10

Les assistants vocaux développés sur Windows 10 doivent appliquer les directives d’expérience utilisateur ci-dessous pour fournir les meilleures expériences d’activation vocale sur Windows 10. Ce document guide les développeurs tout au long des tâches clés à effectuer pour intégrer un assistant vocal au shell Windows 10.

## <a name="contents"></a>Contents

- [Récapitulatif des vues d’activation vocale prises en charge dans Windows 10](#summary-of-voice-activation-views-supported-in-windows-10)
- [Récapitulatif des exigences](#requirements-summary)
- [Bonnes pratiques en vue d’une expérience d’écoute attentive](#best-practices-for-good-listening-experiences)
- [Guide de conception de l’activation vocale dans l’application](#design-guidance-for-in-app-voice-activation)
- [Guide de conception de l’activation vocale au-dessus du verrouillage](#design-guidance-for-voice-activation-above-lock)
- [Guide de conception de l’aperçu de l’activation vocale](#design-guidance-for-voice-activation-preview)

## <a name="summary-of-voice-activation-views-supported-in-windows-10"></a>Récapitulatif des vues d’activation vocale prises en charge dans Windows 10

Windows 10 déduit l’expérience d’activation pour le contexte du client en fonction du contexte de l’appareil. Le tableau suivant fournit une vue d’ensemble des différentes vues disponibles quand l’écran est allumé.

| Vue (disponibilité) | Contexte d’appareil | Objectif du client | Visible quand | Besoins de conception |
| --- | --- | --- | --- | --- |
| **Dans l’application (19H1)** | En dessous du verrouillage, l’assistant a le focus | Interagir avec l’application de l’assistant | L’assistant traite la demande dans l’application | Expérience d’écoute dans la vue principale de l’application |
| **Au-dessus du verrouillage (19H2)** | Au-dessus du verrouillage, utilisateur non authentifié | Interagir avec l’assistant, mais à distance | Le système est verrouillé et l’assistant demande l’activation | Visuels plein écran pour l’interface utilisateur en champ lointain. Implémenter des stratégies de masquage pour ne pas bloquer le déverrouillage. |
| **Aperçu de l’activation vocale (20H1)** | Au-dessous du verrouillage, l’assistant n’a pas le focus | Interagir avec l’assistant, mais de manière moins intrusive | Le système est en dessous du verrouillage et l’assistant demande l’activation en arrière-plan | Canevas minimal. Transférer à la vue de l’application principale au besoin ou la redimensionner. |

## <a name="requirements-summary"></a>Récapitulatif des exigences

Un effort minimal est nécessaire pour accéder aux différentes expériences. Toutefois, les assistants doivent implémenter les guides de conception propres à chaque vue. Le tableau ci-dessous dresse la liste des exigences à respecter.

| **Vue de l’activation vocale** | **Récapitulatif des exigences au niveau de l’assistant** |
| --- | --- |
| **Dans l’application** | <ul><li>Traiter la demande dans l’application</li><li>Fournit des indicateurs d’interface utilisateur pour les états d’écoute</li><li>L’interface utilisateur s’adapte à la modification des tailles de fenêtre</li></ul> |
| **Au-dessus du verrouillage** | <ul><li>Détecter l’état du verrouillage et demander l’activation</li><li>Ne pas fournir une expérience utilisateur toujours persistante qui bloque l’accès à l’écran de verrouillage de Windows</li><li>Fournir des visuels plein écran et une expérience vocale en premier</li><li>Respecter les conseils de masquage ci-dessous</li><li>Suivre les considérations relatives à la confidentialité et à la sécurité ci-dessous</li></ul> |
| **Aperçu de l’activation vocale** | <ul><li>Détecter l’état de déverrouillage et demander l’activation en arrière-plan</li><li>Dessiner une expérience utilisateur d’écoute minimale dans le volet d’aperçu</li><li>Dessiner dans le coin supérieur droit un X de fermeture qui, une fois actionné, masque automatiquement et arrête la diffusion en streaming de l’audio</li><li>Transférer à la vue principale de l’application de l’assistant ou la redimensionner au besoin pour fournir des réponses</li></ul> |

## <a name="best-practices-for-good-listening-experiences"></a>Bonnes pratiques en vue d’une expérience d’écoute attentive

L’assistant doit créer une expérience d’écoute capable de communiquer au client des informations critiques sur son état. Voici quelques états possibles à considérer lors de la création d’une expérience d’assistant. Ce sont là des suggestions, pas des obligations.

- L’assistant accepte les entrées vocales
- L’assistant est en cours d’activation (par mot clé ou pression sur le bouton du micro)
- L’assistant diffuse en streaming des données audio vers le cloud de l’assistant
- L’assistant est prêt à écouter le client parler
- L’assistant est à l’écoute des mots prononcés
- L’assistant comprend que le client a fini de parler
- L’assistant traite et prépare une réponse
- L’assistant répond

Même si les états changent rapidement, songez à fournir une expérience utilisateur pour les états car les durées sont variables dans l’écosystème Windows. Les commentaires visuels ainsi que les brefs signaux audio, également appelés &quot;icônes sonores&quot;, peuvent faire partie de la solution. De même, les cartes visuelles associées à des descriptions audio offrent de bonnes options de réponse.

## <a name="design-guidance-for-in-app-voice-activation"></a>Guide de conception de l’activation vocale dans l’application

Quand l’application de l’assistant a le focus, l’intention du client est clairement d’interagir avec l’application. Toutes les expériences d’activation vocale doivent donc être gérées par la vue principale de l’application. Cette vue peut être redimensionnée par le client. Pour mieux vous expliquer les interactions avec le shell de l’assistant, nous allons utiliser l’exemple concret d’un assistant de service financier nommé Contoso dans le reste de ce document. Dans ce diagramme et les suivants, ce que le client dit apparaît dans les bulles de texte situées à gauche et les réponses de l’assistant apparaissent dans les bulles de texte situées à droite.

**Vue dans l’application. État initial au début de l’activation vocale :** 
![capture d’écran montrant l’application d’assistant de service financier Contoso ouverte dans son canevas par défaut. Une bulle de texte à droite indique « Contoso ».](media/voice-assistants/windows_voice_assistant/initial_state.png)

**Vue dans l’application. Après une activation réussie par la voix, l’expérience d’écoute commence :** ![Capture d’écran de l’assistant vocal sur Windows pendant que l’assistant vocal est à l’écoute](media/voice-assistants/windows_voice_assistant/listening.png)

**Vue dans l’application. Toutes les réponses restent dans l’expérience de l’application.** ![Capture d’écran de l’assistant vocal sur Windows pendant que l’assistant répond](media/voice-assistants/windows_voice_assistant/response.png)

## <a name="design-guidance-for-voice-activation-above-lock"></a>Guide de conception de l’activation vocale au-dessus du verrouillage

Disponible avec 19H2, des assistants basés sur la plateforme d’activation vocale Windows sont disponibles pour répondre au-dessus du verrouillage.

### <a name="customer-opt-in"></a>Participation des clients

L’activation vocale au-dessus du verrouillage est toujours désactivée par défaut. Pour participer, les clients doivent sélectionner Paramètres Windows > Confidentialité > Activation vocale. Pour plus d’informations sur la supervision de ce paramètre et les moyens de le demander, consultez le [guide d’implémentation au-dessus du verrouillage](windows-voice-assistants-implementation-guide.md#detecting-above-lock-activation-user-preference).

### <a name="not-a-lock-screen-replacement"></a>Pas de remplacement de l’écran de verrouillage

Bien que des notifications ou d’autres points d’intégration à l’écran de verrouillage pour des applications standard soient disponibles pour l’assistant, l’écran de verrouillage Windows définit toujours l’expérience client initiale jusqu’à ce qu’une activation vocale se produise. Une fois l’activation vocale détectée, l’application de l’assistant apparaît temporairement au-dessus de l’écran de verrouillage. Pour éviter toute confusion chez le client lorsqu’elle est active au-dessus du verrouillage, l’application de l’assistant ne doit jamais présenter d’interface utilisateur pour demander des informations d’identification.

![Capture d’écran d’un écran de verrouillage Windows](media/voice-assistants/windows_voice_assistant/lock_screen1.png)

### <a name="above-lock-experience-following-voice-activation"></a>Expérience au-dessus du verrouillage après l’activation vocale

Quand l’écran est allumé, l’application de l’assistant est en mode plein écran sans barre de titre au-dessus de l’écran de verrouillage. Des visuels plus grands et des descriptions vocales fortes avec une interface vocale principale forte permettent de gérer les cas où le client est trop éloigné pour lire l’interface utilisateur ou a les mains occupées à une autre tâche (non PC).

Quand l’écran est éteint, l’application de l’assistant peut jouer une icône sonore pour indiquer que l’assistant est en cours d’activation et fournir une expérience vocale uniquement.

![Capture d’écran de l’assistant vocal au-dessus du verrouillage](media/voice-assistants/windows_voice_assistant/above_lock_listening.png)

### <a name="dismissal-policies"></a>Stratégies de masquage

L’assistant doit implémenter les conseils de masquage de cette section pour permettre aux clients de se connecter plus facilement la prochaine fois qu’ils souhaitent utiliser leur PC Windows. Voici les exigences spécifiques que l’assistant doit respecter :

- **Tous les canevas de l’assistant qui apparaissent au-dessus du verrouillage doivent contenir un X** dans le coin supérieur droit qui permet de masquer l’assistant.
- **Le fait d’appuyer sur n’importe quelle touche doit également masquer l’application de l’assistant** . L’entrée au clavier est un signal d’application de verrouillage traditionnel qui indique que le client souhaite se connecter. Par conséquent, toute entrée clavier/texte ne doit pas être dirigée vers l’application. Au lieu de cela, l’application doit être automatiquement masquée si une entrée au clavier est détectée pour permettre au client de se connecter facilement à son appareil.
- **Si l’écran est éteint, l’application doit se masquer automatiquement.** De cette façon, la prochaine fois que le client utilise son PC, l’écran de connexion est prêt.
- Si l’application est « en cours d’utilisation », elle peut continuer à être exécutée au-dessus du verrouillage. « En cours d’utilisation » constitue toute entrée ou sortie. Par exemple, lors de la diffusion en streaming de morceaux de musique ou de vidéos, l’application peut continuer au-dessus du verrouillage. Le &quot;suivi&quot; et d’autres étapes de dialogues multitours sont autorisés à conserver l’application au-dessus du verrouillage.
- Vous trouverez des **détails sur l’implémentation du masquage de l’application** dans le [guide d’implémentation au-dessus du verrouillage](windows-voice-assistants-implementation-guide.md#closing-the-application).

![Capture d’écran montrant la vue de verrouillage ci-dessus de l’application d’assistant de service financier Contoso.](media/voice-assistants/windows_voice_assistant/above_lock_response.png)

![Capture d’écran d’un appareil de bureau montrant l’écran de verrouillage Windows.](media/voice-assistants/windows_voice_assistant/lock_screen2.png)

### <a name="privacy-amp-security-considerations-above-lock"></a>Considérations relatives à la confidentialité et à la sécurité au-dessus du verrouillage

Si de nombreux PC sont portables, ils ne sont pas toujours sous le contrôle de leur propriétaire. Ils peuvent ainsi être brièvement laissés sans assistance dans des chambres d’hôtel, sur des sièges d’avion ou dans des espaces de travail auxquels d’autres personnes ont accès. Si les assistants activés au-dessus du verrouillage ne sont pas préparés, ils peuvent faire l’objet d’une classe d’attaque appelée « [evil maid](https://en.wikipedia.org/wiki/Evil_maid_attack) » (femme de chambre malveillante).

Les assistants doivent donc suivre les conseils de cette section pour garantir la sécurité de l’expérience. L’interaction au-dessus du verrouillage se produit quand l’utilisateur Windows n’est pas authentifié. En général, cela signifie que la **saisie de texte dans l’assistant doit également être traitée comme non authentifiée** .

- Les assistants doivent **implémenter une liste verte de compétences pour identifier celles qui sont confirmées comme sécurisées** , ce qui signifie qu’elles sont accessibles au-dessus du verrouillage.
- Les technologies d’ID de l’orateur peuvent jouer un rôle dans la réduction de certains risques, mais l’ID de l’orateur n’est pas un remplacement approprié pour l’authentification Windows.
- La liste verte de compétences doit prendre en compte trois classes d’actions ou de compétences :

| **Classe d’action** | **Description** | **Exemples (liste non exhaustive)** |
| --- | --- | --- |
| Sans danger sans authentification | Informations générales ou commande et contrôle de l’application de base | « Quelle heure est-il ? », « Lire la piste suivante » |
| Sans danger avec ID d’orateur | Risque d’emprunt d’identité révélant des informations personnelles. | « Quel est mon prochain rendez-vous ? », « Consulter la liste de mes achats », « Répondre à l’appel » |
| Sans danger uniquement après l’authentification Windows | Actions à haut risque qu’un attaquant peut utiliser pour nuire au client | « Acheter d’autres produits d’alimentation », « Supprimer mon rendez-vous (important) », « Envoyer un message texte (malveillant) », « Lancer une page web (néfaste) » |

Dans le cas de Contoso, les informations générales sur les actions publiques sont sans danger sans authentification. Les informations spécifiques au client telles que le nombre d’actions détenues sont probablement sans danger avec l’ID de haut-parleur. Toutefois, l’achat ou la vente d’actions ne doit jamais être autorisé sans authentification Windows.

Pour sécuriser davantage l’expérience, **les liens web ou autres lancements inter-applications sont toujours bloqués par Windows jusqu’à ce que le client se connecte** . En dernier recours, Microsoft se réserve le droit de supprimer une application de la liste verte des assistants activés si un problème de sécurité sérieux n’est pas traité dans un délai convenable.

## <a name="design-guidance-for-voice-activation-preview"></a>Guide de conception de l’aperçu de l’activation vocale

En dessous du verrouillage, quand l’application de l’assistant _n’a pas_ le focus, Windows fournit une interface utilisateur d’activation vocale moins intrusive pour ne pas déranger le flux du client. Cela est particulièrement vrai dans le cas de fausses activations qui pourraient être très perturbatrices si elles lançaient l’application complète. L’idée principale est que chaque assistant a une autre page d’accueil dans le shell, l’icône de barre des tâches de l’assistant. Quand la demande d’activation en arrière-plan se produit, une petite vue au-dessus de l’icône de la barre des tâches de l’assistant apparaît. Les assistants doivent fournir une petite expérience d’écoute dans ce canevas. Une fois les demandes traitées, les assistants peuvent choisir de redimensionner cette vue pour afficher une réponse contextuelle ou de transférer la vue principale de leur application pour afficher des visuels plus grands et plus détaillés.

- Pour rester minime, l’aperçu n’a pas de barre de titre. **L’assistant doit donc dessiner un X dans le coin supérieur droit pour permettre aux clients de masquer la vue.** Pour savoir quelles API spécifiques appeler quand le bouton de masquage est activé, consultez [Fermeture de l’application](windows-voice-assistants-implementation-guide.md#closing-the-application).
- Pour prendre en charge les aperçus de l’activation vocale, les assistants peuvent inviter les clients à épingler l’assistant à la barre des tâches durant la première exécution.

**Aperçu de l’activation vocale : état initial**

L’assistant Contoso a une icône circulaire en forme de tourbillon dans la barre des tâches.

![Capture d’écran de l’assistant vocal sur Windows sous la forme d’une icône dans la barre des tâches avant l’activation](media/voice-assistants/windows_voice_assistant/pre_compact_view.png)

**À mesure que l’activation progresse** , l’assistant demande l’activation en arrière-plan. L’assistant reçoit un petit volet d’aperçu (par défaut, d’une largeur de 408 et d’une hauteur de 248). Si l’activation vocale côté serveur détermine que le signal est un faux positif, cette vue peut être masquée pour minimiser l’interruption.

![Capture d’écran de l’assistant vocal sur Windows en mode compact pendant la vérification de l’activation](media/voice-assistants/windows_voice_assistant/compact_view_activating.png)

**Une fois l’activation finale confirmée** , l’assistant présente son expérience utilisateur d’écoute. L’assistant doit toujours dessiner une un X de masquage en haut à droite de l’aperçu de l’activation vocale.

![Capture d’écran de l’assistant vocal sur Windows à l’écoute en mode compact](media/voice-assistants/windows_voice_assistant/compact_view_listening.png)

Des **réponses rapides** peuvent être présentées dans l’aperçu de l’activation vocale. Un TryResizeView permet aux assistants de demander des tailles différentes.

![Capture d’écran de l’assistant vocal sur Windows envoyant une réponse en mode compact](media/voice-assistants/windows_voice_assistant/compact_view_response.png)

**Transfert** . À tout moment, l’assistant peut passer à sa vue d’application principale pour fournir plus d’informations, une boîte de dialogue ou des réponses qui nécessitent davantage d’espace à l’écran. Pour plus d’informations sur l’implémentation, consultez la section [Transition de la vue compacte vers la vue complète](windows-voice-assistants-implementation-guide.md#transition-from-compact-view-to-full-view).

![Captures d’écran de l’assistant vocal sur Windows avant et après le développement du mode compact](media/voice-assistants/windows_voice_assistant/compact_transition.png)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Bien démarrer avec le développement de votre assistant vocal](how-to-windows-voice-assistants-get-started.md)