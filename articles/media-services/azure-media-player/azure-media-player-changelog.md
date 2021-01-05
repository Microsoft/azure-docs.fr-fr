---
title: Journal des modifications du Lecteur multimédia Azure
description: Journal des modifications du Lecteur multimédia Azure.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: overview
ms.date: 09/23/2020
ms.openlocfilehash: 9d126f591da1bffebbfd29119c56927cfbb21dcd
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92370574"
---
# <a name="changelog"></a>Journal des modifications

## <a name="236-official-update-september-21-2020"></a>2.3.6 (mise à jour officielle du 21 septembre 2020)

### <a name="features-236"></a>Fonctionnalités 2.3.6

Ajout du support audio uniquement pour la technologie azureHtml5JS (DASH) Prise en charge du début tardif de transcription en direct Prise en charge du changement de langue dans la transcription en direct

### <a name="bug-fixes-236"></a>Correctifs de bogues 2.3.6

Lors de l’utilisation de « playsinline » dans les lectures HLS sur les appareils Apple, le fait de cliquer sur le bouton « LIVE » entraîne le redémarrage de la vidéo L’image de l’affiche AMP provoque parfois une exception Le bouton de volume était manquant lors d’une lecture FairPlay HLS [Accessibilité] Info-bulles non définies pour les boutons lorsque le clavier est utilisé [Accessibilité] Le ratio de luminosité est inférieur à 1,3:1 pour la barre de progression [Accessibilité] Le focus clavier ne retourne parfois pas au bouton de qualité vidéo [Accessibilité] Les contrôles ne sont pas visibles sur l’écran vidéo et le Narrateur ne parvient pas à les trouver

### <a name="changes-236"></a>Modifications 2.3.6

Retourner les erreurs significatives de remise de clé aux applications appelantes

## <a name="235-official-update-june-1-2020"></a>2.3.5 (mise à jour officielle du 1er juin 2020)

### <a name="bug-fixes-235"></a>Correctifs de bogues 2.3.5

- [Accessibilité] L’écouteur de la touche Échap dans le volet Options est attaché au document
- [Accessibilité] Empêcher la disparition de l’interface utilisateur du lecteur si la barre de contrôle ou le menu Options contient le focus
- La barre de contrôle affiche un temps horloge incorrect quand les paramètres d’affichage du temps horloge sont activés

### <a name="changes-235"></a>Modifications 2.3.5

- Ajout et documentation d’un message d’erreur pour le code d’erreur 0x00400005

## <a name="234-official-update-march-4-2020"></a>2.3.4 (mise à jour officielle du 4 mars 2020)

### <a name="bug-fixes-234"></a>Correctifs de bogues 2.3.4

- Impossible de définir PlayReady overrideLicenseAcquistionUrl
- Impossible de lire du contenu avec des discontinuités
- [Accessibilité] La valeur de l’attribut ID pour l’alerte du lecteur d’écran doit être unique
- [Accessibilité] Lors de la navigation dans la boîte de dialogue de paramètres Sous-titres, le focus se déplace hors de la boîte de dialogue

### <a name="changes-234"></a>Modifications 2.3.4

- Journaliser Content-Length après un téléchargement réussi pour faciliter l’analyse des erreurs de déchiffrement 2.3.3 (mise à jour officielle du 12 novembre 2019)

### <a name="features-234"></a>Fonctionnalités 2.3.4

- Ajout de la prise en charge de l’affichage du temps horloge d’une vidéo sous forme de superposition et dans la barre de contrôle

### <a name="bug-fixes-234"></a>Correctifs de bogues 2.3.4

- Le commutateur de piste audio fonctionne, mais génère sur IE 11 et Windows 7 l’erreur « L’objet ne gère pas la propriété ou la méthode 'enabled' »
- Le commutateur de piste audio échoue quand le tampon est entièrement chargé
- Le commutateur de piste audio échoue quand l’utilisateur interrompt la vidéo et bascule entre les pistes audio très rapidement
- [Accessibilité] Info-bulles non définies pour le contrôle vidéo sous le lecteur vidéo
- Boutons de volume manquants sur HTML5 selon le moment auquel « loadstart » est reçu
- [Accessibilité] Impossible de définir le texte de remplacement pour l’image d’affiche
- [Accessibilité] Le focus de l’application est perdu après la sélection de l’option « Terminé » dans la boîte de dialogue des paramètres des légendes
- [Accessibilité] Des attributs ARIA incorrects sont définis pour « video » sous « segments preview »

### <a name="changes-234"></a>Modifications 2.3.4

- Piste/étiquette de légende vide supprimée lors de la lecture de HLS sur iOS et macOS Safari
- Réduction du nombre d’erreurs 412 pour les légendes IMSC1
- Générer un avertissement dans la console pour 10 réponses de légende IMSC1 vides consécutives pour faciliter le débogage en direct

## <a name="232-official-update-october-9-2019"></a>2.3.2 (mise à jour officielle du 9 octobre 2019)

### <a name="features-232"></a>Fonctionnalités 2.3.2

-Ajout de la prise en charge de PlayReady pour la lecture DASH pour le navigateur Chromium Edge

### <a name="bug-fixes-232"></a>Correctifs de bogues 2.3.2

- La vitesse de lecture actuelle n’est pas affichée dans le menu Vitesse de lecture, sauf si l’utilisateur la définit manuellement
- [Accessibilité] Le volet « Paramètres » n’est pas réduit avec la touche Échap
- [Accessibilité] La touche de raccourci AMP « M » ne fonctionne pas quand le narrateur est activé

### <a name="changes-232"></a>Modifications 2.3.2

- Pour les navigateurs qui ne prennent pas en charge le codec audio E-AC3, les pistes audio E-AC3 sont masquées dans le menu Piste audio.
- Pour les navigateurs qui prennent en charge le codec audio E-AC3, une piste audio E-AC3 est sélectionnée par défaut.
- Pour les navigateurs qui ne prennent pas en charge le changement de codec audio, les pistes audio dont le codec diffère de celui de la piste sélectionnée sont masquées dans le menu Piste audio.

## <a name="231-official-update-august-12-2019"></a>2.3.1 (mise à jour officielle du 12 août 2019)

### <a name="features-231"></a>Fonctionnalités 2.3.1

- Signaler un événement lors de la réception de zones de message pendant une lecture DASH - Ajout de la prise en charge de l’affichage des pistes audio ce-3 dans le menu Audio sur les navigateurs qui prennent en charge ce-3 et de la possibilité de passer d’une piste audio aac à ec3 et vice versa uniquement sur le navigateur Edge basé sur Chromimum

### <a name="bug-fixes-231"></a>Correctifs de bogues 2.3.1

- Le menu Piste audio est endommagé après la suppression des pistes ec-3
- La durée actuelle peut être supérieure à la durée de la vidéo
- La définition de la vitesse de lecture via initialSpeed ne fonctionne pas
- Parfois, après une recherche, le lecteur semble coincé
- Sur Edge et IE, sur un écran tactile, après avoir effectué un zoom sur une page, le fait d’appuyer ou de pointer sur la barre de recherche ne permet pas d’obtenir avec précision le segment correct de la vidéo
- [Accessibilité] L’étiquette Aria pour la lecture/pause n’est pas explicite pour le lecteur vidéo, mise en correspondance de l’erreur de segment actif introuvable pour flashSS avec la bonne erreur AMP
- [Accessibilité] Les rôles Aria utilisés pour la lecture/pause doivent être conformes aux valeurs valides (.vjs-text-track-display)
- [Accessibilité] Certains rôles ARIA doivent se trouver dans des parents particuliers
- [Accessibilité] Aucune info-bulle n’est définie pour le bouton de lecture/pause du lecteur vidéo. Les légendes IMSC1 peuvent disparaître après une recherche dans la mémoire tampon audio/vidéo actuelle.

### <a name="changes-231"></a>Modifications 2.3.1

- Lors de l’obtention d’une erreur segmentDecryptError et si le lecteur se trouve dans la périphérie en direct, le lecteur actualise le manifeste au lieu d’essayer le segment suivant.
- Enrichissement de la journalisation pour le diagnostic
- Mise à jour de la documentation afin d’inclure la prise en charge de FairPlay pour iOS Safari
- Ajout d’un exemple pour le « srclang » de l’option IMSC1
- Ajout des remplacements padding, textPadding et boxShadow pour les pistes de texte.
- Ajout d’un code d’erreur (0x0020025B) pour bien indiquer que l’échec du téléchargement du segment est dû à l’absence de connexion Internet, au lieu de simplement lever une erreur 0x00200259

## <a name="230-official-release-april-30-2019"></a>2.3.0 (publication officielle du 30 avril 2019)

### <a name="features-230"></a>Fonctionnalités 2.3.0

- Ajout de la prise en charge des légendes IMSC1 pour DASH
- Ajout de la prise en charge des ressources vidéo uniquement pour DASH
- Ajout de l’API presentationTimeOffsetInSec

### <a name="bug-fixes-230"></a>Correctifs de bogues 2.3.0

- Le profil d’heuristiques AMP LowLatency interfère avec la lecture vidéo iOS. Les paramètres d’activation (« mute ») et de désactivation (« unmute ») du son pour certaines langues ont des traductions erronées
- La valeur aria-valuenow du curseur de la barre de progression est parfois incorrecte
- La valeur de rôle aria de l’affichage de la piste de texte est incorrecte

### <a name="changes-230"></a>Modifications 2.3.0

- Les journaux incluent maintenant la taille des fragments multimédias téléchargés
- Suppression de la prise en charge d’IE 9 et d’IE 10
- Mise à jour de l’exemple CEA708 pour illustrer l’alignement à gauche des légendes
- Inclusion de MediaError.message dans les journaux pour les échecs de lecture

## <a name="224-official-update-february-22-2019"></a>2.2.4 (mise à jour officielle du 22 février 2019) ##

### <a name="bug-fixes-224"></a>Correctifs de bogues 2.2.4 ###

- [Correctif de bogue][AMP][Accessibilité] Suppression d’un onglet fantôme accessible lors de l’apparition de l’écran d’erreur
- [Correctif de bogue][AMP] Correction de la touche d’accès rapide « M » pour Internet Explorer 11 et Edge
- [Correctif de bogue][AMP] Correction d’une exception pour les sous-titres CEA708
- [Correctif de bogue][AMP] Résolution d’un problème de blocage vidéo pour le navigateur Edge

### <a name="changes-224"></a>Modifications 2.2.4 ###

- [Modification][AMP] Lors dela production d’une erreur de déchiffrement de fragment, le lecteur réessaye divers fragments actuels pour récupérer la lecture
- [Modification][AMP] Plus grande tolérance de AMP au chevauchement de fragments vidéo ou audio

## <a name="223-official-update-january-9-2019"></a>2.2.3 (mise à jour officielle du 9 janvier 2019) ##

### <a name="features-223"></a>Fonctionnalités 2.2.3 ###

- [Fonctionnalité][HLS] Ajout du menu de pistes audio pour la lecture HLS de Safari

### <a name="bug-fixes-223"></a>Correctifs de bogues 2.2.3 ###

- [Correctif de bogue][AMP][Accessibilité] Impossibilité de sélectionner le bouton « Live » à l’aide du clavier lors de lectures de diffusion en direct
- [Correctif de bogue][AMP] Correction des erreurs 0x0400003 fausses positives en raison de l’échec du test MSE
- [Correctif de bogue][AMP] Résolution d’un problème où la vidéo pouvait se bloquer au démarrage d’un stream en direct

### <a name="changes-223"></a>Modifications 2.2.3 ###

- [Modification][AMP] Ajout d’informations supplémentaires dans le journal pour permettre de meilleurs diagnostics
- [Modification][AMP] Lorsque plusieurs vitesses de transmission sont disponibles dans la même résolution d’écran, celles-ci sont toutes disponibles à la sélection

## <a name="222-official-update"></a>2.2.2 (mise à jour officielle) ##

### <a name="bug-fixes-222"></a>Correctifs de bogues 2.2.2 ###

- [Correctif de bogue][AMP] Lorsque le lecteur rencontre une panne temporaire du réseau, la lecture est immédiatement arrêtée
- [Correctif de bogue][AMP][Accessibilité] La boîte de dialogue d’erreur n’est pas accessible à l’aide du clavier
- [Correctif de bogue][AMP] Plutôt qu’une erreur non prise en charge, affichage d’une boucle de progression infinie lors de la lecture d’un actif multimédia uniquement audio

### <a name="changes-222"></a>Modifications 2.2.2 ###

- [Modification][AMP] Ajout de chaînes localisées pour l’interface utilisateur de publication

## <a name="221-official-update"></a>2.2.1 (mise à jour officielle) ##

### <a name="features-221"></a>Fonctionnalités 2.2.1 ###

- [Fonctionnalité][CMAF] Ajout de la prise en charge de HLS CMAF

### <a name="bug-fixes"></a>Correctifs de bogues ###

- [Correctif de bogue][AMP] Minuteurs non effacés dans la logique de nouvelle tentative, ce qui génère des erreurs de lecture
- [Correctif de bogue][AMP][Firefox] L’événement terminé n’est pas déclenché sur Firefox et sur Chrome lorsque le programme en direct est arrêté
- [Correctif de bogue][AMP] Affichage de contrôles après setsource, même lorsque ces contrôles ont la valeur false dans les options du lecteur

### <a name="changes"></a>Modifications ###

- [Modification][Sous-titrage en direct] Changement du nom de l’API pour les sous-titres CEA de 608 à 708. Pour plus d’informations, consultez [Paramètres des sous-titres CEA708](/javascript/api/azuremediaplayer/cea708captionssettings)-->.

## <a name="220-official-release"></a>2.2.0 (publication officielle) ##

### <a name="features-220"></a>Fonctionnalités 2.2.0 ###

- [Fonctionnalité][Azurehtml5JS][Flash][Sous-titres en direct] Prise en charge du sous-titrage CEA 708 dans les technologies Azurehtml5JS et FlashSS pour le contenu Clear et AES.

### <a name="bug-fixes-220"></a>Correctifs de bogues 2.2.0 ###

- [Correctif de bogue] La détection de la version Flash ne fonctionne pas dans Chrome/Edge

### <a name="changes-220"></a>Modifications 2.2.0 ###

- [Modification][AMP][Heuristique] Remplacement du nom du profil heuristique QuickStart par LowLatency
- [Modification][Flash] Modification dans Flash Player pour que la détection de version active la lecture du contenu AES avec la nouvelle mise à jour d’Adobe Flash.

## <a name="219-official-hotfix"></a>2.1.9 (correctif officiel) ##

### <a name="bug-fixes-219"></a>Correctifs de bogues 2.1.9 ###

- [Correctif de bogue][En direct] Exception survenant lors de la transition de streams en direct aux archives de vidéo à la demande/en direct

### <a name="changes-219"></a>Modifications 2.1.9 ###

- [Modification][Flash][AES] Modification de la logique de la technologie Flash, afin de ne pas utiliser sharedbytearrays pour le déchiffrement AES, car Adobe a bloqué l’utilisation depuis Flash 30. Notez que la lecture ne fonctionne qu’après le déploiement d’une nouvelle version de Flash par Adobe, en raison d’un bogue dans v30. Pour plus de détails, voir [Problèmes connus](azure-media-player-known-issues.md).

## <a name="218-official-update"></a>2.1.8 (mise à jour officielle) ##

### <a name="bug-fixes-218"></a>Correctifs de bogues 2.1.8 ###

- [Correctif de bogue] Parfois la boucle de progression n’affiche pas la recherche ni la pré-lecture
- [Correctif de bogue] Le lecteur ne démarre pas en mode silencieux lorsque l’option Muet est activée
- [Correctif de bogue] Affichage du curseur de volume lorsque les contrôles ont la valeur false
- [Correctif de bogue] Lecture parfois répétée quand l’utilisateur passe à la session en direct
- [Correctif de bogue][Firefox] Le lecteur lève parfois une exception JavaScript au chargement
- [Correctif de bogue][Accessibilité] Perte du contour du focus par les boutons Lire/Pause/Volume quand ils sont sélectionnés à l’aide des contrôles du clavier
- [Correctif de bogue] Suppression de la fuite de mémoire fixe sur le lecteur
- [Correctif de bogue] L’appel de src() après une erreur de lecteur ne réinitialise pas la source
- [Correctif de bogue][En direct] AMP est en état de chargement constant lorsque l’utilisateur clique sur le bouton Live après la fin de la diffusion
- [Correctif de bogue][Chrome] Blocage du lecteur et échec de lecture lorsque le navigateur est réduit en arrière-plan.

### <a name="changes-218"></a>Modifications 2.1.8 ###

- [Modification] Mise à jour de l’erreur 0x0600001 à afficher quand le contenu AES est lu avec Flash 30, car ce plug-in n’est pas pris en charge pour le moment. Pour plus de détails, voir [Problèmes connus](azure-media-player-known-issues.md).
- [Modification] Ajout de nouvelles tentatives supplémentaires pour les scénarios en direct quand le manifeste demande 404 ou retourne des manifestes vides.

## <a name="217-official-update"></a>2.1.7 (mise à jour officielle) ##

### <a name="features-217"></a>Fonctionnalités 2.1.7 ###

- [Fonctionnalité][AzureHtml5JS] Ajout de l’option de configuration pour vider les données obsolètes dans la mémoire tampon source du média

### <a name="bug-fixes-217"></a>Correctifs de bogues 2.1.7 ###

- [Correctif de bogue][Accessibilité][Lecteur d’écran] Suppression de l’en-tête vide que le lecteur incluait lorsque le titre n’était pas défini
- [Correctif de bogue][UWA] AMP lève une exception lors de la lecture dans une application Windows universelle
- [Correctif de bogue][OSX] setActiveTextTrack() ne fonctionne pas dans Safari sur OSx
- [Correctif de bogue][En Direct] Cliquer sur la session en direct après la suppression et la réinitialisation du lecteur génère une exception
- [Correctif de bogue][Apparence] Heure actuelle tronquée pour certains actifs multimédias
- [Correctif de bogue][DRM] Correctif inclus pour prendre en charge la lecture dans les navigateurs qui gèrent plusieurs DRM CENC

### <a name="changes-217"></a>Modifications 2.1.7 ###

- [Modification][Exemples][Accessibilité] Ajout de la balise de langue à tous les exemples

## <a name="216-official-update"></a>2.1.6 (mise à jour officielle) ##

### <a name="bug-fixes-216"></a>Correctifs de bogues 2.1.6 ###

- [Correctif de bogue] Affichage AMP indiquant une durée incorrecte pour un actif multimédia spécifique
- [Correctif de bogue][FairPlay-HLS] Non-propagation d’erreurs Fairplay à l’interface utilisateur
- [Correctif de bogue] Propriétés de l’heuristique personnalisée actuellement ignorées dans AMP 2.1.5.

### <a name="changes-216"></a>Modifications 2.1.6 ###

- [Modification][DRM FairPlay] Suppression du délai d’expiration de la demande de certificat et de celle de licence pour FairPlay, afin de conserver la parité avec les implémentations PlayReady et Widevine
- [Modification] Améliorations heuristiques diverses pour remédier au contenu flou

### <a name="features-216"></a>Fonctionnalités 2.1.6 ###

- [Fonctionnalité] Ajout de la prise en charge du format mpd-time-cmaf

## <a name="215-official-hotfix"></a>2.1.5 (correctif officiel) ##

### <a name="bug-fixes-215"></a>Correctifs de bogues 2.1.5 ###

- [Correctif de bogue][Sous-titres] Le style VTT n’est pas restitué correctement par le lecteur
- [Correctif de bogue][Accessibilité] Le bouton Live n’a pas d’étiquette Aria

## <a name="214-official-update"></a>2.1.4 (mise à jour officielle) ##

### <a name="bug-fixes-214"></a>Correctifs de bogues 2.1.4 ###

- [Correctif de bogue][Accessibilité][Focus] Les utilisateurs ne peuvent pas se servir de la touche Tab pour mettre le focus sur les boutons personnalisés qui sont ajoutés à droite du bouton Plein écran dans la barre de contrôle
- [Correctif de bogue][IE11][Barre de volume] L’utilisation de la touche de tabulation pour atteindre la fenêtre contextuelle du volume fait clignoter tout l’écran vidéo dans Internet Explorer 11 en mode plein écran
- [Correctif de bogue][Apparence|Flush] Affichage d’un espace entre la barre de contrôle et la barre de volume
- [Correctif de bogue][AMP][Sous-titres] Les anciennes pistes incorporées ne sont pas effacées lorsque la source est modifiée sur un lecteur existant
- [Correctif de bogue][Accessibilité][Narrateur] Lecture incorrecte du contrôle du volume par le lecteur d’écran
- [Correctif de bogue][FlashSS] Parfois l’événement de lecture ne se déclenche pas à partir de la technologie Flash
- [Correctif de bogue][AMP][Focus] Le bouton Lire/Pause nécessite deux clics quand le lecteur a le focus et qu’il est en mode plein écran
- [Correctif de bogue][AMP][Apparence] Affichage d’une durée incorrecte dans la barre de progression d’un actif multimédia spécifique
- [Correctif de bogue][Publicités][Pub Butler] L’analyseur VAST ne gère pas le fichier VAST qui n’a pas d’événement de progression
- [Correctif de bogue][SDN][AMP 2.1.1] Résolution du problème de prise en charge du plug-in SDN Hive
- [Correctif de bogue][Accessibilité] Le Narrateur lit « Bouton Midnight Mute » lorsque l’utilisateur a le focus sur le bouton du volume

### <a name="changes-214"></a>Modifications 2.1.4 ###

- [Modification][Accessibilité][Technologie d’assistance] Les boutons disposent désormais d’une propriété aria-live pour améliorer l’expérience avec la technologie d’assistance
- [Modification][Accessibilité][Bouton Volume|Narrateur] Amélioration de l’accessibilité du bouton de volume en modifiant la fonctionnalité de tabulation et le comportement du curseur. Ces modifications permettent aux utilisateurs du clavier de modifier plus facilement le volume du lecteur.
- [Modification] Délai d’expiration du menu contextuel d’inactivité passé de 3 à 5 secondes
- [Modification][Accessibilité][Luminosité] Amélioration du rapport contraste-luminosité dans les menus déroulants des paramètres de sous-titres

## <a name="213-official-update"></a>2.1.3 (mise à jour officielle) ##

### <a name="bug-fixes-213"></a>Correctifs de bogues 2.1.3 ###

- [Correctif de bogue][Plug-ins|Superposition de titre] Le plug-in de superposition de titre lève des exceptions JS avec AMP v2.X+
- [Correctif de bogue] Envoi de l’événement Source Set à la console JavaScript même lorsque la journalisation est désactivée
- [Correctif de bogue][Apparence] Les info-bulles de temps du lecteur sont affichées en dehors du contexte du lecteur quand vous pointez sur l’une des extrémités de la barre de durée
- [Correctif de bogue][Accessibilité][Lecteur d’écran] Le Narrateur lit « Region Landmark » ou « Video Player Region Landmark » lorsque l’utilisateur a le focus sur le lecteur
- [Correctif de bogue][AMP] Désactivation impossible du mode lecteur via CSS
- [Correctif de bogue][Accessibilité] Impossible d’utiliser la tabulation pour placer le focus sur tout le lecteur quand l’utilisateur est en mode plein écran
- [Correctif de bogue][Apparence][En direct] L’apparence ne répond pas au texte LIVE localisé en japonais
- [Correctif de bogue][Apparence] La durée et l’heure actuelle sont coupées lorsque le flux > 60 min - [Correctif de bogue][iPhone|En direct] Affichage par le lecteur du texte pour l’heure actuelle ou la durée dans la barre de contrôle
- [Correctif de bogue][AMP] L’appel des API heuristiques du lecteur génère des exceptions JavaScript
- [Correctif de bogue][Html5 natif|iOS] Non propagation de la propriété Videotag « playsinline » au lecteur
- [Correctif de bogue][iOS|iframe] Passage du lecteur au mode plein écran impossible sur iPhone si le lecteur est chargé dans un iframe
- [Correctif de bogue][AMP][Heuristique] AMP fonctionne toujours avec le profil hybride sans tenir compte des options du lecteur
- [Correctif de bogue][AMP|Win8.1] Levée d’une exception lors de l’hébergement dans l’application Win8.1 avec une vue web

### <a name="changes-213"></a>Modifications 2.1.3 ###

- [Modification][AMP] Ajout d’informations sur le point de terminaison CDN dans l’événement FragmentDownloadComplete
- [Modification][AMP][En direct] Latence de streaming en direct améliorée et optimisée

## <a name="212-official-hotfix"></a>2.1.2 (correctif officiel) ##

### <a name="bug-fixes-212"></a>Correctifs de bogues 2.1.2 ####

- [Correctif de bogue][Accessibilité][Narrateur de Windows] Le Narrateur lit « Progress midnight » lorsque l’utilisateur a le contexte de la barre de progression et que l’heure actuelle est 0:00
- [Correctif de bogue][Apparence] Codage en dur de la taille du logo en code JavaScript
- [Accessibilité][Touches d’accès rapide] Non activation des touches d’accès rapide lorsque l’utilisateur clique sur le lecteur.

### <a name="changes-212"></a>Modifications 2.1.2 ####

- [Modification][Journalisation] Enregistrement de l’URL du manifeste en cas d’échec de chargement du manifeste par le lecteur

### <a name="features-212"></a>Fonctionnalités 2.1.2 ###

- [Modification][Performance][Optimisation] Amélioration des temps de chargement et de démarrage du lecteur

## <a name="211-official-update"></a>2.1.1 (mise à jour officielle) ##

### <a name="bug-fixes-211"></a>Correctifs de bogues 2.1.1 ####

- [Correctif de bogue][iOS] La définition de la lecture automatique sur false génère une boucle de progression infinie dans Safari pour iOS
- [Correctif de bogue] La recherche d’une heure supérieure à la durée du contenu génère une boucle de progression infinie
- [Correctif de bogue] Les touches d’accès rapide demandent plusieurs tabulations au clavier pour que le contexte du lecteur fonctionne
- [Correctif de bogue] Dans certains actifs multimédias, la vidéo se fige quelques secondes après le redimensionnement du lecteur
- [Correctif de bogue] Boucle de progression infinie (après la recherche) lorsque l’utilisateur effectue plusieurs recherches rapidement
- [Correctif de bogue] La barre de contrôle n’est pas masquée pendant l’inactivité
- [Correctif de bogue] L’ouverture d’une application web qui héberge AMP peut entraîner le chargement de la page web à deux reprises
- [Correctif de bogue] Boucle infinie lors de la lecture du contenu de certains actifs multimédias via la technologie Flash
- [Correctif de bogue] Absence d’affichage du menu Autres options avec les plug-ins tiers
- [Correctif de bogue][Apparence|Tube][En direct] Affichage de deux icônes Live lorsque le lecteur se trouve dans la session en direct d’un programme
- [Correctif de bogue][Apparence] Désactivation du logo impossible
- [Correctif de bogue][Contenu DD+] Apparition d’une boucle de progression continue pour les actifs multimédias contenant une piste audio Dolby Digital
- [Correctif de bogue] Blocage du panneau AMP le plus récent lors du basculement des pistes de langue audio pendant le stream en direct
- [Correctif de bogue] Disparition de l’arrière-plan fixe pour une boucle de progression
- [Correctif de bogue] Corrections des bogues de boucle de progression infinie dans les exemples statiques de jeton Flash AES

### <a name="changes-211"></a>Modifications 2.1.1 ####

- [Modification] Ajout du code d’erreur pour la spécification Https Widevine : à partir de Chrome v58, le contenu Widevine doit être chargé/lu via le protocole `https://`, sinon la lecture échoue.
- [Modification] Ajout de l’étiquette Aria pour le chargement de la boucle de progression, afin que la technologie d’assistance puisse commenter « chargement de la vidéo » lorsque du contenu est chargé  

## <a name="210-official-release"></a>2.1.0 (publication officielle) ##

### <a name="features-210"></a>Fonctionnalités 2.1.0 ###

- [Fonctionnalité][AzureHtml5JS] Prise en charge de publicités VOD en début, à mi- et en fin de bandes
- [Fonctionnalité][Bêta][AzureHtml5JS] Prise en charge de publicités en direct, en début à-mi et en fin de bandes
- [Fonctionnalité] Ajout d’une option de nouvelle apparence - AMP-flush
- [Fonctionnalité] Ajout d’étiquettes Aria améliorées pour une meilleure intégration aux lecteurs d’écran/à la technologie d’assistance
- [Fonctionnalité][Apparence] L’apparence affiche désormais clairement toutes les icônes et tous les boutons en mode de contraste élevé

### <a name="bug-fixes-210"></a>Correctifs de bogues 2.1.0 ###

- [Correctif de bogue] Ensemble de correctifs se rapportant à l’interface utilisateur et à l’accessibilité
- [Correctif de bogue] Chargement incorrect de AMP dans Internet Explorer 9

### <a name="changes-210"></a>Modifications 2.1.0 ###

- [Modification] Éléments DOM restructurés dans le lecteur pour prendre en charge les publicités
- [Modification] Passage de CSS à SCSS pour le développement de l’apparence
- [Modification][Exemples] Ajout d’un exemple pour les publicités VOD
- [Modification][Exemples] Ajout d’un exemple pour la vitesse de lecture
- [Modification][Exemples] Ajout d’un exemple pour l’apparence Flush

## <a name="200-beta-release"></a>2.0.0 (version bêta) ##

- [Modification] Mise à jour vers VJS5
- [Fonctionnalité] Ajout d’une nouvelle API fluide pour une réactivité agile du lecteur
- [Fonctionnalité] Vitesse de lecture
- [Modification] Passage de CSS à SCSS pour l’apparence

## <a name="183-official-hotfix-update"></a>1.8.3 (mise à jour du correctif officiel) ##

### <a name="bug-fixes-183"></a>Correctifs de bogues 1.8.3 ###

- [Correctif de bogue][AzureHtml5JS] La lecture de certains actifs multimédias ayant un DTS négatif est impossible dans Chrome

## <a name="182-official-hotfix-update"></a>1.8.2 (mise à jour du correctif officiel) ##

### <a name="bug-fixes-182"></a>Correctifs de bogues 1.8.2 ###

- [Correctif de bogue][AzureHtml5JS] Les vitesses de transmission audio supérieures ne sont pas lues via AzureHtml5JS

## <a name="181-official-update"></a>1.8.1 (mise à jour officielle) ##

### <a name="bug-fixes-181"></a>Correctifs de bogues 1.8.1 ###

- [Correctif de bogue][iOS] Absence d’affichage des légendes/sous-titres dans le lecteur natif
- [Correctif de bogue][AMP] Les URL de streaming sauvegardées par CDN et comportant des jetons d’authentification ne sont pas lues
- [Correctif de bogue][FairPlay] Absence d’ID Tech (Bits [31-28] du code d’erreur) dans le code d’erreur FairPlay (consultez les codes d’erreur pour plus de détails)
- [Correctif de bogue][Safari][PlayReady] Contenu PlayReady dans Safari produisant une boucle de progression infinie

### <a name="changes-181"></a>Modifications 1.8.1 ###

- [Modification][Html5] Modification des journaux détaillés de la technologie Html5 native pour contenir des événements à partir de VideoTag

## <a name="180-official-update"></a>1.8.0 (mise à jour officielle) ##

### <a name="features-180"></a>Fonctionnalités 1.8.0 ###

- [Fonctionnalités][DRM] Ajout de la prise en charge de FairPlay (voir [Contenu protégé](azure-media-player-protected-content.md) pour plus d’informations)

### <a name="bug-fixes-180"></a>Correctifs de bogues 1.8.0 ###

- [Correctif de bogue][AMP] La recherche de l’utilisateur ne déclenche pas d’événement d’attente lorsque le réseau est limité
- [Correctif de bogue][FlashSS] Levée d’une exception par la sélection d’une qualité dans la technologie Flash
- [Correctif de bogue][AMP] La sélection dynamique d’une qualité s’affiche réellement dans le menu contextuel
- [Correctif de bogue][Apparence] Difficulté de sélection du dernier élément de menu des menus contextuels

### <a name="changes-180"></a>Modifications 1.8.0 ###

- [Modification] Mise à jour du lecteur avec les exigences EME Chrome actuelles
- [Modification] Modification de l’ordre des technologies par défaut pour prendre en compte la nouvelle technologie html5FairPlayHLS (pour plus d’informations, consultez [Contenu protégé](azure-media-player-protected-content.md))
- [Modification][AzureHtml5JS] Activation de la lecture MPEG-Dash dans Safari
- [Modification][Exemples] Modification des exemples Multi-DRM pour prendre en charge FairPlay

## <a name="174-official-hotfix-update"></a>1.7.4 (mise à jour du correctif officiel) ##

### <a name="bug-fixes-174"></a>Correctifs de bogues 1.7.4 ###

- [Correctif de bogue][Chrome] Apparition d’un contour bleu autour de la poignée de recherche lorsque l’utilisateur a le contexte du lecteur
- [Correctif de bogue][IE9] Levée d’une exception JavaScript lorsque le lecteur est chargé dans Internet Explorer 9

## <a name="173-official-hotfix-update"></a>1.7.3 (mise à jour du correctif officiel) ##

### <a name="bug-fixes-173"></a>Correctifs de bogues 1.7.3 ###

- [Correctif de bogue][AzureHtml5JS] Expiration du délai d’attente du lecteur dans les réseaux limités

### <a name="changes-173"></a>Modifications 1.7.3 ###

- [Modification] Activation de Webcrypto sur Edge pour le déchiffrement du contenu AES
- [Modification] Optimisation de l’heuristique AMP pour tenir compte des blocs mis en cache
- [Modification][AzureHtml5JS] Optimisation de l’heuristique en réduisant la latence d’estimation de la bande passante

## <a name="172-official-hotfix-update"></a>1.7.2 (mise à jour du correctif officiel) ##

### <a name="features-172"></a>Fonctionnalités 1.7.2 ###
<!---API needs onboarding. Removed link to API until remedied.--->
- [Fonctionnalité][AzureHtml5JS|Firefox] Activation de la lecture Widevine avec EME pour Firefox 47+
- [Fonctionnalité] Ajout d’un événement pour la suppression du lecteur
<!-- ([disposing](index.html#static-amp.eventname.disposing)) -->

### <a name="bug-fixes-172"></a>Correctifs de bogues 1.7.2 ###

- [Correctif de bogue] Décodage incorrect des paramètres de requête d’URL CDN Akamai codés
- [Correctif de bogue] Levée d’exception en cours sur manifestPlayableWindowLength()
- [Correctif de bogue] À la fin de la vidéo l’utilisateur ne peut pas toujours cliquer sur Lire pour la revoir
- [Correctif de bogue] Dimensionnement réactif non conforme aux changements rapides de taille de fenêtre
- [Correctif de bogue][Edge|IE] Pas de prise en compte du dimensionnement réactif au chargement de la page pour width=x, height=auto
- [Correctif de bogue][Android|Chrome] Demande d’autorisations de Chrome pour lire le contenu DRM lorsque le contenu n’est pas chiffré
- [Correctif de bogue][Accessibilité][Edge] Sélection incorrecte par les contrôles de clavier des éléments de menu contextuel
- [Correctif de bogue][Accessibilité] Absence d’affichage de bordure en mode de contraste élevé
- [Correctif de bogue][FlashSS] La non-suppression de l’écouteur de l’événement mouse up après la suppression du lecteur provoque une exception
- [Correctif de bogue][FlashSS] Problème rencontré lors de l’analyse de l’URL du manifeste avec des espaces codés
- [Correctif de bogue][iOS] Erreur de type lors de l’évaluation de tech.featuresVolumeControl

### <a name="changes-172"></a>Modifications 1.7.2 ###

- [Modification][DRM] Contrôles DRM déplacés après la définition de la source pour ne vérifier que lorsque le contenu est chiffré
- [Modification][AES] Suppression du corps non défini de type/plain à partir de la requête de remise de clé
- [Modification][Accessibilité] Désormais le Narrateur de Windows lit « Lecteur multimédia » quand le contexte se trouve sur le lecteur et non sur les propriétés

## <a name="171-official-hotfix-update"></a>1.7.1 (mise à jour du correctif officiel) ##

### <a name="features-171"></a>Fonctionnalités 1.7.1 ###

- [Fonctionnalité] Ajout d’option pour le profil Heuristique hybride (ce profil est défini par défaut)

### <a name="bug-fixes-171"></a>Correctifs de bogues 1.7.1 ###

- [Correctif de bogue] La conception dynamique ne fonctionne pas selon la norme HTML5 (largeur=100 %, hauteur=auto)
- [Correctif de bogue] Les valeurs en pourcentage de la largeur et de la hauteur ne se comportent pas comme prévu dans v1.7.0

## <a name="170-official-update"></a>1.7.0 (mise à jour officielle) ##

### <a name="features-170"></a>Fonctionnalités 1.7.0 ###
<!---API needs onboarding. Removed link until remedied.--->
- [Fonctionnalité][AzureHtml5JS][FlashSS] Ajout de currentMediaTime() pour obtenir le temps du média encodeur de l’heure actuelle en secondes
- [Fonctionnalité][FlashSS] Implémentation du téléchargement des API de télémétrie avec videoBufferData() et audioBufferData()<!-- (see [BufferData](index.html#amp.bufferdata) for more details) -->
- [Fonctionnalité][FlashSS] Ajout de l’événement « downloadbitratechanged »
- [Fonctionnalité] Temps de chargement amélioré par rapport aux versions antérieures du lecteur
- [Fonctionnalité] Les erreurs sont consignées dans la console JavaScript

### <a name="bug-fixes-170"></a>Correctifs de bogues 1.7.0 ###

- [Correctif de bogue] L’URL d’affiche codée avec des paramètres de chaîne de requête ne s’affiche pas dans le lecteur
- [Correctif de bogue] Levée d’exception lorsque aucune technologie n’est chargée et que l’API amp.Player.poster() est appelée
- [Correctif de bogue] Levée d’exception lorsque des fonctions essaient d’accéder au lecteur après sa suppression
- [Correctif de bogue][Accessibilité] Absence de contour sur le focus placé sur l’en-tête de recherche de la barre de progression
- [Correctif de bogue][Accessibilité] Présence d’une ombre sur les menus contextuels en mode de contraste élevé
- [Correctif de bogue][iOS] La lecture des sous-titres WebVTT du lecteur natif ne fonctionne pas
- [Correctif de bogue][AzureHtml5JS] L’erreur 0x0100002 devrait s’afficher lors de la diffusion d’un flux HTTP sur un site HTTPS qui génère à la place une boucle de progression infinie comme résultat d’un contenu mixte
- [Correctif de bogue][AzureHtml5JS] Absence de segment de fin provoquant une erreur de contrôle d’intégrité en boucle qui affiche une perception d’état infini de la mise en mémoire tampon
- [Correctif de bogue][AzureHtml5JS] Nom de piste audio incorrect dans le menu lorsque useManifestForLabel=false et que des codes de langue de trois lettres sont utilisés
- [Correctif de bogue][AzureHtml5JS|Chrome] Perception d’état infini de la mémoire tampon à la fin du contenu en raison d’une imprécision de virgule flottante dans la durée avec JavaScript dans Chrome
- [Correctif de bogue][FlashSS] Affichage momentané d’une erreur intermittente récupérable lors de la création de Flash Player
- [Correctif de bogue][FlashSS] Échec de la lecture lorsque les flux vidéo et audio utilisent des échelles de temps différentes en raison de l’imprécision de l’arrondi qui échoue avec le message « URL de fragment (...) n’a pas pu générer FLVTags »
- [Correctif de bogue][FlashSS] Problèmes rencontrés lors de l’analyse d’URL de manifeste avec des espaces codés
- [Correctif de bogue][FlashSS] Absence de vérification permettant de déterminer si c’est la version de Flash Player >= 11.4 qui provoque une erreur de lecture au lieu de revenir à la technologie suivante dans l’ordre des technologies
- [Correctif de bogue][FlashSS][AES] Problèmes rencontrés lors de l’acceptation de jetons AES comportant des traits de soulignement
- [Correctif de bogue][SilverlightSS|OSX] « // » servant de préfixe à un manifeste au lieu du protocole (HTTP ou HTTPS) est reconnu comme fichier local générant une boucle de progression infinie

### <a name="changes-170"></a>Modifications 1.7.0 ###

- [Modification][FlashSS] Scripts SWF fusionnés (« MSAdaptiveStreamingPlugin-osmf2.0.swf » et « StrobeMediaPlayback.2.0.swf ») en un seul fichier SWF appelé « StrobeMediaPlayback.2.0.swf »
- [Modification][FlashSS] Mise à jour de la propagation de codes d’erreur pour obtenir des codes d’erreur plus précis (par exemple, les 404 génèrent désormais 0x30200194 au lieu de l’erreur générique 0x30200000)

## <a name="163-official-hotfix-update"></a>1.6.3 (mise à jour du correctif officiel) ##

### <a name="bug-fixes-163"></a>Correctifs de bogues 1.6.3 ###

- [Correctif de bogue] Exception d’exécution JavaScript lorsque le gestionnaire d’événements des touches d’accès rapide est exécuté après la suppression du lecteur
- [Correctif de bogue][Android][AzureHtml5JS] Pas de lecture sur un appareil mobile utilisant un réseau de téléphonie mobile
- [Correctif de bogue] Mise à jour de Forge afin qu’il s’exécute en tant que rôle de travail pour libérer l’interface utilisateur

## <a name="162-official-hotfix-update"></a>1.6.2 (mise à jour du correctif officiel) ##

### <a name="features-162"></a>Fonctionnalités 1.6.2 ###

- [Fonctionnalité] Ajout de langues supplémentaires pour la localisation (consultez la documentation pour plus de détails)

### <a name="bug-fixes-162"></a>Correctifs de bogues 1.6.2 ###

- [Correctif de bogue][IE9-10] Le fait de cliquer sur des zones autour du lecteur réduisait la fenêtre du navigateur à cause du bogue dans Internet Explorer 9/Internet Explorer 10 qui réduit sur window.blur()
- [Correctif de bogue][FlashSS] Refus des jetons AES comportant des traits de soulignement

## <a name="161-official-hotfix-update"></a>1.6.1 (mise à jour du correctif officiel) ##

### <a name="bug-fixes-161"></a>Correctifs de bogues 1.6.1 ###

- [Correctif de bogue][FlashSS|Edge,IE][SilverlightSS|IE] Impossible d’obtenir le focus sur d’autres éléments d’interface utilisateur pour les entrées, etc. dans Internet Explorer/Edge
- [Correctif de bogue] Échec de la lecture AES lorsque Forge n’est pas défini
- [Correctif de bogue][Android][AzureHtml5JS|Chrome] Pas de lecture de contenu de la boucle de progression continue pendant une boucle de contrôle de l’intégrité
- [Correctif de bogue][IE9] Pas de prise en charge de console.log() par Internet Explorer 9 ce qui déclenche une exception

## <a name="160-official-update"></a>1.6.0 (mise à jour officielle) ##

### <a name="features-160"></a>Fonctionnalités 1.6.0 ###

- [Fonctionnalité] Réduction de 33 % de la taille de azuremediaplayer.min.js
- [Fonctionnalité] [AzureHtml5JS|Edge][Non testé] Prise en charge des flux audio DD+ dans Edge (pas de changement de codec après le choix initial). L’application doit sélectionner le flux audio approprié à ce stade.
- [Fonctionnalité] Contrôles de touche d’accès rapide (pour plus d’informations, consultez la documentation)
- [Fonctionnalité] Info-bulle de la durée de progression par pointage de la souris pour une recherche précise du temps
- [Fonctionnalité] Possibilité de détection asynchrone de plug-ins si la méthode setupDone existe dans le plug-in

### <a name="bug-fixes-160"></a>Correctifs de bogues 1.6.0 ###

- [Correctif de bogue] Pas de vidage sur getMemoryLog(true) du journal mémoire
- [Correctif de bogue] La zone de sélection de la vitesse de transmission se réinitialise au déplacement de la souris ce qui provoque un problème de sélection des vitesses inférieures avec le contrôle de la souris
- [Correctif de bogue] Plantage de Mac Office dans l’application lors de l’exécution de la vérification DRM
- [Correctif de bogue] Les classes CSS sont facilement remplacées par hasard
- [Correctif de bogue][Chrome] Mise à jour de l’identification à partir de la chaîne de l’agent utilisateur dans le navigateur Edge
- [Correctif de bogue][AzureHtml5JS] Absence du bouton des sous-titres dans la barre d’outils de Edge(Win10) ou Chrome(Mac)
- [Correctif de bogue][Android][AzureHtml5JS|Chrome] Exception InvalidStateError à l’appel de endOfStream() sur de courtes vidéos
- [Correctif de bogue][Firefox] Suppression de l’avertissement DRM provoqué par Firefox lors de la vérification des fonctionnalités du navigateur
- [Correctif de bogue][Html5] Absence d’affichage des Sous-titres/Légendes avec du contenu MP4 progressif
- [Correctif de bogue][FlashSS] Les messages avec des horodatages correspondants ont été consignés dans l’ordre inverse
- [Correctif de bogue][Accessibilité][Chrome|Firefox] La touche Tab et les contrôles de sélection sélectionnent automatiquement le premier élément de menu
- [Correctif de bogue][Accessibilité] Touche Tab pour contrôler le bouton de volume

### <a name="changes-160"></a>Modifications 1.6.0 ###

- [Modification] Utilisation du temps de déchiffrement AES pour la sélection du niveau de qualité
- [Modification] Mise à jour de la réécriture d’URL afin d’utiliser HLS v4 avant HLS v3 pour les flux audio multiples
- [Modification] Définition de nativeControlsForTouch sur false comme valeur par défaut (doit avoir la valeur false pour fonctionner correctement)

## <a name="150-official-update"></a>1.5.0 (mise à jour officielle) ##

### <a name="features-150"></a>Fonctionnalités 1.5.0 ###

- [Fonctionnalité] Améliorations de la sécurité web générale (prévention d’injection, XSS, etc.)
- [Fonctionnalité] Crochets d’intégration du plug-in SDN pour l’événement sourceset et options.sdn
- [Fonctionnalité] Gestion de la robustesse des erreurs 5XX et 4XX pendant la lecture

### <a name="bug-fixes-150"></a>Correctifs de bogues 1.5.0 ###

- [Correctif de bogue] Mise à jour de la minimisation CSS afin d’utiliser des codes de polices d’entités HTML pour les boutons au lieu d’Unicode
- [Correctif de bogue][AzureHtml5JS] Sélection systématique par le contenu multi-DRM du jeton du premier élément dans protectionInfo, ce qui provoque l’échec de la deuxième DRM
- [Correctif de bogue][AzureHtml5JS] La recherche ne se termine jamais lors d’une recherche dans une zone comportant des segments manquants.
- [Correctif de bogue][AzureHtml5JS|Edge] Activation de la norme EME préfixée dans la mise à jour Edge pour la lecture PlayReady
- [Correctif de bogue][AzureHtml5JS|Firefox] Mise à jour de la vérification EME pour autoriser Firefox v42+ (avec MSE) à revenir vers Silverlight pour le contenu protégé
- [Correctif de bogue][FlashSS] Mise à jour du message d’erreur pour passer du numéro à la chaîne détaillée

### <a name="changes-150"></a>Modifications 1.5.0 ###

- [Modification] Les affiches ne fonctionnent actuellement que sous forme d’URL absolues.

## <a name="140-official-update"></a>1.4.0 (mise à jour officielle) ##

### <a name="features-140"></a>Fonctionnalités 1.4.0 ###

- [Fonctionnalité][AzureHtml5JS|Chrome] Prise en charge de Simple DRM Widevine
- [Fonctionnalité][AzureHtml5JS] Gestion de la robustesse des erreurs 404/412 pendant la lecture

### <a name="bug-fixes-140"></a>Correctifs de bogues 1.4.0 ###

- [Correctif de bogue][FlashSS] Amélioration de la validation des paramètres

## <a name="130-official-update"></a>1.3.0 (mise à jour officielle) ##

### <a name="features-130"></a>Fonctionnalités 1.3.0 ###

- [Fonctionnalité][AzureHtml5JS][FlashSS] Commutation audio du même contenu multi-audio de codec

### <a name="bug-fixes-130"></a>Correctifs de bogues 1.3.0 ###

- [Correctif de bogue][AzureHtml5JS|Chrome] Boucle de protection infinie intermittente
- [Correctif de bogue][AzureHtml5JS|IE][Windows Phone] Exception provoquant des problèmes de lecture de Windows Phone
- [Correctif de bogue][FlashSS] Échec de la définition de l’exécution automatique sur false pour les instances supplémentaires
- [Correctif de bogue] Problèmes de dimensionnement des menus de l’interface utilisateur

## <a name="120-official-update"></a>1.2.0 (mise à jour officielle) ##

### <a name="features-120"></a>Fonctionnalités 1.2.0 ###

- [Fonctionnalité][AzureHtml5JS|Firefox] Prise en charge lorsque la norme MSE est activée
- [Fonctionnalité] L’application n’a plus besoin de fournir de chemins pour les binaires de la technologie de secours (swf, xap). Le chemin est relatif au script du Lecteur multimédia Azure.

### <a name="bug-fixes-120"></a>Correctifs de bogues 1.2.0 ###

- [Correctif de bogue][AzureHtml5JS|Chrome] Dérives du lecteur derrière la session en direct lorsque le lecteur se trouve en arrière-plan
- [Correctif de bogue][AzureHtml5JS|Edge] Le mode plein écran ne fonctionne pas
- [Correctif de bogue][AzureHtml5JS] La journalisation n’a pas été activée correctement lors de sa définition dans les options
- [Correctif de bogue][Flash] La « mise en mémoire tampon » et l’icône associée apparaissent pendant l’événement d’attente
- [Correctif de bogue] Autorisation de la lecture à se poursuivre en cas d’échec de la demande de bande passante initiale
- [Correctif de bogue] Échec de chargement du lecteur en cas d’initialisation avec des options non définies
- [Correctif de bogue] Production d’une exception vdata lors de la tentative de suppression du lecteur alors qu’il a déjà été supprimé
- [Correctif de bogue] Mappage incorrecte des icônes de la barre de qualité

## <a name="111-official-hotfix-update"></a>1.1.1 (mise à jour du correctif officiel) ##

### <a name="bug-fixes-111"></a>Correctifs de bogues 1.1.1 ###

- [Correctif de bogue] Ancien problème de mode plein écran dans Internet Explorer
- [Correctif de bogue] Les plug-ins ne sont plus remplacés

## <a name="110-official-update"></a>1.1.0 (mise à jour officielle) ##

### <a name="features-110"></a>Fonctionnalités 1.1.0 ###

- [Fonctionnalité] Mise à jour des chaînes de localisation de l’interface utilisateur

### <a name="bug-fixes-110"></a>Correctifs de bogues 1.1.0 ###

- [Correctif de bogue] Contraste insuffisant du gros bouton Lire
- [Correctif de bogue] Indicateur visuel du focus sur tabulation
- [Correctif de bogue] Utilisation d’informations de résolution désormais correctes lors de la sélection du menu Vitesse de transmission
- [Correctif de bogue] Menu Autres options désormais dimensionné dynamiquement
- [Correctif de bogue] Divers problèmes d’interface utilisateur

## <a name="100-official-release"></a>1.0.0 (publication officielle) ##

### <a name="features-100"></a>Fonctionnalités 1.0.0 ###

- [Fonctionnalité] Test d’accessibilité de base pour le contrôle Tab, le contrôle de focus, le lecteur d’écran, l’interface utilisateur à contraste élevé
- [Fonctionnalité] Mise à jour de l’interface utilisateur
- [Fonctionnalité] Journalisation Dév.
- [Fonctionnalité] API pour la définition dynamique des pistes des légendes/sous-titres
- [Fonctionnalité] Fonctionnalités de localisation de base
- [Fonctionnalité] Consolidation du code d’erreur entre les technologies
- [Fonctionnalité] Nouveau code d’erreur lorsque les plug-ins (tels que Flash ou Silverlight) ne sont pas installés
- [Fonctionnalité][AzureHtml5JS] Implémentation d’événements de diagnostic de base

### <a name="bug-fixes-100"></a>Correctifs de bogues 1.0.0 ###
<!---What is that actually supposed to say?--->
- [Correctif de bogue][AzureHtml5JS] La lecture en direct se fige sur les mises à jour MPD lorsqu’il existe de petites imprécisions dans l’horodatage
- [Correctif de bogue][AzureHtml5JS] Atténuation de plusieurs problèmes de lecture en direct
- [Correctif de bogue][AzureHtml5JS] Vidage des mémoires tampons lorsque l’heuristique de la taille des fenêtres est activée et accède à un écran de résolution supérieure
- [Correctif de bogue][AzureHtml5JS] Chrome affiche désormais correctement l’événement terminé. Est lié au problème connu antérieur selon lequel *Chrome n’envoie pas correctement l’événement terminé lorsque AzureHtml5JS est utilisé. Il y a un problème dans le navigateur sous-jacent.*
- [Correctif de bogue][AzureHtml5JS] Désactivation de Safari pour cette technologie afin de résoudre le *Problème de lecture avec OSX Yosemite et la technologie AzureHtml5JS. Il existe des problèmes d’implémentation de MSE. Atténuation temporaire : forcer flashSS, silverlightSS comme ordre des technologies pour ces agents utilisateurs*
- [Correctif de bogue][FlashSS] Déclenchement de loadstart après la survenue d’une erreur

## <a name="020-beta"></a>0.2.0 (Bêta) ##

### <a name="features-020"></a>Fonctionnalités 0.2.0 ###

- [Fonctionnalité] Test terminé concernant PlayReady et AES pour des événements à la demande et en direct - voir la matrice de compatibilité
- [Fonctionnalité] Gestion des discontinuités
- [Fonctionnalité] Prise en charge des horodatages supérieurs à 2^53
- [Fonctionnalité] Persistance du paramètre de requête d’URL dans la requête du manifeste
- [Fonctionnalité][Non testé] Prise en charge des profils d’heuristique `QuickStart` et `HighQuality`
- [Fonctionnalité][Non testé] Exposition d’informations de flux vidéo pour les vitesses de transmission, la largeur et la hauteur sur AzureHtml5JS et FlashSS
- [Fonctionnalité][Non testé] Sélection de la vitesse de transmission sur AzureHtml5JS et FlashSS (voir la documentation de l’API)

### <a name="bug-fixes-020"></a>Correctifs de bogues 0.2.0 ###

- [Correctif de bogue] Gros bouton de lecture désormais visible sur WP8.1
- [Correctif de bogue] Correction de plusieurs problèmes de lecture en direct
- [Correctif de bogue] Le bouton de son désactivé fonctionne maintenant dans l’interface utilisateur
- [Correctif de bogue] Mise à jour de l’expérience de chargement de l’interface utilisateur pour le mode Lecture automatique
- [Correctif de bogue] Problème de chargeur AMD et définition de conflits de méthode
- [Correctif de bogue] Problème de chargement de l’application WP 8.1 Cordova
- [Correctif de bogue] ProtectionType pris en charge par la plateforme/technologie de requêtes de contenu protégé afin de sélectionner la technologie appropriée pour la lecture.  Résolution du problème antérieur connu que le «  _contenu PlayReady sur chrome (bureau) / Safari 8 (sur OSX Yosemite) ne revient pas sur le lecteur Silverlight_  »
- [Correctif de bogue] Exception non interceptée sur WinServer 2012 R2 étant donné que Media Foundation n’est pas installé sur cette machine par défaut.  Tentative d’utilisation d’API de balises vidéo HTML qui ne sont pas implémentées, ce qui génère une erreur. L’atténuation actuelle consiste à intercepter cette erreur et à retourner false au lieu de lever l’erreur.
- [Correctif de bogue] Obtention systématique du segment init après une recherche ou un échec http pour éviter les erreurs lors de la lecture
- [Correctif de bogue] Désactivation du suivi de la progression simulée et des mises à jour de l’heure lors de la survenue de l’erreur.
- [Correctif de bogue] Suppression du menu contextuel
- [Correctif de bogue][AzureHtml5JS] Absence d’affichage du message d’erreur lors de la définition d’un jeton non valide pour le contenu PlayReady
- [Correctif de bogue][AzureHtml5JS] Pas de prise en compte de l’heuristique de la taille des fenêtres lors du passage au plein écran pendant la lecture en direct
- [Correctif de bogue] [FlashSS] Suppression des messages publiés de Strobe Media Player afin que seuls les messages du Lecteur multimédia Azure s’affichent
- [Correctif de bogue][SilverlightSS] Pas de récupération de l’événement « recherché » quand une recherche au-delà de la durée ou inférieure à 0 est effectuée

## <a name="010-beta-release"></a>0.1.0 (version bêta) ##

Préversion initiale

## <a name="next-steps"></a>Étapes suivantes ##

- [Démarrage rapide du Lecteur multimédia Azure](azure-media-player-quickstart.md)
