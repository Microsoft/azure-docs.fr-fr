---
title: Enregistrer des échantillons vocaux personnalisés - Service Speech
titleSuffix: Azure Cognitive Services
description: Créez une voix personnalisée de qualité production en préparant un script complet, en recrutant des voix professionnelles et en effectuant un enregistrement professionnel.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/13/2020
ms.author: erhopf
ms.openlocfilehash: dae7b8e0485c1a2456b85e0910f60b2164d4e41c
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95026316"
---
# <a name="record-voice-samples-to-create-a-custom-voice"></a>Enregistrer des échantillons vocaux pour créer une voix personnalisée

La création d’une voix personnalisée de qualité production à partir de zéro n’est pas une tâche épisodique. Une voix personnalisée se compose avant tout d’une grande collection d’échantillons audio de voix humaine. Il est essentiel que ces enregistrements audio soient de haute qualité. Choisissez une voix professionnelle ayant une expérience dans ces types d’enregistrements, et enregistrez-la en faisant appel à un ingénieur spécialisé disposant d’un matériel professionnel.

Mais avant de pouvoir effectuer ces enregistrements, vous avez besoin d’un script, c’est-à-dire les mots qui seront énoncés par votre voix professionnelle pour créer les échantillons audio. Pour de meilleurs résultats, votre script doit avoir une bonne couverture phonétique et être suffisamment varié pour former le modèle vocal personnalisé.

L’enregistrement d’une voix professionnelle implique de nombreux petits détails très importants. Ce guide est une feuille de route détaillant le processus qui vous aidera à obtenir des résultats homogènes.

> [!TIP]
> Pour optimiser au maximum ces résultats, n’hésitez pas à contacter Microsoft, qui vous aidera à développer votre voix personnalisée. Microsoft possède une grande expérience dans la production de voix de haute qualité pour ses propres produits, y compris Cortana et Office.

## <a name="voice-recording-roles"></a>Rôles lors d’un enregistrement vocal

Un projet d’enregistrement d’une voix personnalisée implique quatre rôles de base :

Role|Objectif
-|-
Voix professionnelle        |La voix de cette personne servira de base à la voix personnalisée.
Ingénieur du son  |Surveille les aspects techniques de l’enregistrement et commande le matériel d’enregistrement.
Directeur            |Prépare le script et dirige l’enregistrement de la voix professionnelle.
Éditeur              |Finalise les fichiers audio et prépare leur transfert vers le portail de la voix professionnelle.

Une même personne peut remplir plusieurs rôles. Ce guide part du principe que vous tiendrez principalement le rôle de directeur et que recruterez la voix professionnelle et l’ingénieur du son. Si vous souhaitez effectuer vous-même les enregistrements, cet article fournit certaines informations sur le rôle d’ingénieur du son. Le rôle Éditeur n’est nécessaire qu’après la session, et peut donc être rempli par le directeur ou l’ingénieur du son.

## <a name="choose-your-voice-talent"></a>Choisir votre voix professionnelle

Les acteurs spécialisés dans la voix off ou le doublage sont de bons candidats. Vous trouverez également de bons profils parmi les annonceurs et les présentateurs.

Choisissez une personne dont la voix naturelle vous plaît. Il est possible de créer une voix de « personnage », unique, mais il est beaucoup plus difficile pour la plupart des acteurs de reproduire cette voix de manière cohérente, et cet effort risque d’entraîner une déformation de la voix.

> [!TIP]
> En règle générale, évitez d’utiliser des voix facilement reconnaissables pour créer une voix personnalisée, à moins, bien sûr, que votre objectif est de reproduire une voix célèbre. Les voix plus anonymes sont mieux acceptées par les utilisateurs.

Le principal facteur quant au choix de la voix professionnelle est l’homogénéité. Vos enregistrements doivent donner l’impression qu’ils ont été réalisés le même jour et dans la même pièce. Pour tendre à cet idéal, vous devez suivre de bonnes pratiques en matière d’enregistrement et d’ingénierie.

Votre voix professionnelle représente l’autre moitié de l’équation. Le débit, le volume, la hauteur et la tonalité de la voix doivent être réguliers. Une parfaite diction est indispensable. L’acteur doit aussi pouvoir contrôler de manière stricte ses variations de tonalité, ses émotions ou ses tics de langage.

Les enregistrements d’échantillons de voix personnalisées peuvent être plus fatigants que d’autres types d’exercices vocaux. La plupart des voix professionnelles peuvent effectuer des sessions d’enregistrement de deux ou trois heures par jour. Limitez-vous à trois ou quatre sessions par semaine, en accordant si possible un jour de repos entre deux sessions.

Les enregistrements effectués pour un modèle de voix doivent être émotionnellement neutres. Autrement dit, un énoncé triste ne doit pas être lu de manière triste. Un ton particulier peut être ajouté à la synthèse vocale ultérieurement via des contrôles de prosodie. Travaillez avec votre acteur pour créer un « personnage » qui définit le ton global et le niveau d’émotion de la voix personnalisée. Dans ce processus, vous allez identifier les sons « neutres » de ce personnage.

Par exemple, un personnage peut avoir une personnalité naturellement chaleureuse. Par conséquent, « sa » voix peut véhiculer une note d’optimisme même quand il parle de façon neutre. Mais un tel trait de caractère doit rester subtil et homogène. Écoutez des énoncés de voix existantes pour avoir une idée de ce que vous recherchez.

> [!TIP]
> En règle générale, vous devez garder la propriété des enregistrements vocaux que vous effectuez. Votre acteur devrait travailler sous contrat dans le cadre de votre projet.

## <a name="create-a-script"></a>Créer un script

Le point de départ de toute session d’enregistrement de voix personnalisée est le script, qui contient les énoncés qui seront lus par votre acteur. (Le terme « énoncés » englobe les phrases complètes et les expressions plus courtes.)

Les énoncés de votre script peuvent provenir de n’importe quelle source : fiction, non-fiction, transcription de discours, bulletin d’informations ou tout autre document imprimé. Si vous souhaitez vous assurer que votre voix est parfaitement adaptée à l’énoncé de mots spécifiques (par exemple, la terminologie médicale ou le jargon de la programmation), vous pouvez inclure des phrases tirées d’ouvrages spécialisés ou de documents techniques. Pour obtenir une brève description des potentiels problèmes juridiques, consultez la section [« Aspects juridiques »](#legalities). Vous pouvez également écrire votre propre texte.

Vos énoncés ne doivent pas nécessairement provenir de la même source ou du même type de source. Et leurs contenus peuvent être de nature totalement différente. Toutefois, si vous comptez utiliser des expressions spécifiques (par exemple, « Vous vous êtes connecté avec succès ») dans votre application de reconnaissance vocale, veillez à les inclure dans votre script. Vous aiderez ainsi votre acteur à bien prononcer ces expressions. Et si vous décidez d’utiliser un enregistrement à la place d’une synthèse vocale, vous aurez déjà ces expressions dans la même voix.

L’homogénéité est essentielle dans le choix d’une voix professionnelle, mais la variété est le signe d’un bon script. Votre script doit inclure un grand nombre de mots et de phrases variées, avec un large éventail de longueurs, de structures et de tons. Chaque son dans la langue doit être représenté plusieurs fois dans de nombreux contextes (c’est la *couverture phonétique).*

En outre, le texte doit inclure toutes les représentations d’un son particulier par écrit et placer chaque son à différents endroits dans les phrases. Les phrases déclaratives et les questions doivent être incluses et lues avec l’intonation appropriée.

Il est difficile d’écrire un script qui fournit *juste assez* de données pour permettre au portail Custom Speech de générer une bonne voix. En pratique, le moyen le plus simple de créer un script permettant d’obtenir une couverture phonétique efficace consiste à inclure un grand nombre d’échantillons. Les voix standard fournies par Microsoft ont été créées à partir de dizaines de milliers d’énoncés. Vous devez être prêt à enregistrer au moins quelques-uns voire plusieurs milliers énoncés pour générer une voix personnalisée de qualité production.

Vérifiez attentivement que le script ne contient pas d’erreurs. Si possible, demandez aussi à un tiers d’effectuer cette vérification. Lorsque vous parcourez le script avec votre acteur, vous allez probablement trouver d’autres erreurs.

### <a name="script-format"></a>Format de script

Vous pouvez écrire votre script dans Microsoft Word. Ce script sera utilisé lors de la session d’enregistrement et vous pouvez le configurer comme vous le souhaitez. Créez séparément le fichier texte requis par le portail Custom Voice.

Un format de script de base contient trois colonnes :

* Le numéro de l’énoncé, à partir de 1. Cette numérotation aide toutes les personnes dans le studio à accéder directement à un énoncé précis (« reprenons l’énoncé 356 »). Vous pouvez utiliser la fonctionnalité de numérotation de paragraphe de Word pour numéroter automatiquement les lignes du tableau.
* Une colonne vide où vous inscrirez le numéro de la prise ou le code temporel de chaque énoncé, pour vous aider à le repérer dans l’enregistrement terminé.
* Le texte de l’énoncé lui-même.

![Exemple de script](media/custom-voice/script.png)

> [!NOTE]
> La plupart des studios enregistrent de courts segments appelés *prises*. Chaque prise contient généralement de 10 à 24 énoncés. Le simple fait de noter le nombre de prises suffira pour trouver un énoncé plus tard. Si vous enregistrez dans un studio qui souhaite effectuer des enregistrements plus longs, utilisez plutôt des codes temporels. Le studio sera équipé d’un large écran indiquant le code.

Laissez suffisamment d’espace après chaque ligne pour y noter vos remarques. Veillez à ce qu’aucun énoncé n’apparaisse sur deux pages. Numérotez les pages et imprimez votre script sur le recto de chaque page.

Imprimez trois copies du script : une pour l’acteur, une pour l’ingénieur du son et l’autre pour le directeur (vous). Utilisez un trombone plutôt que des agrafes : un acteur expérimenté sépare les pages pour éviter de faire du bruit en les feuilletant.

### <a name="legalities"></a>Aspects juridiques

Conformément à la législation, la lecture par un acteur d’un texte protégé par un copyright est un travail impliquant la rémunération de l’auteur du texte. Ce travail ne sera pas identifiable dans le produit final, la voix personnalisée. Pourtant, le droit d’utiliser un travail protégé par un copyright à cet effet n’est pas bien établi. Microsoft ne peut pas fournir de conseils juridiques à ce sujet ; consultez votre propre conseiller juridique.

Heureusement, il est possible d’éviter totalement ces problèmes. Il existe de nombreuses sources de texte que vous pouvez utiliser sans licence ou autorisation.

|Source du texte|Description|
|-|-|
|[Corpus CMU Arctic](http://festvox.org/cmu_arctic/)|Environ 1 100 phrases sélectionnées provenant d’œuvres libres de droits et destinées spécifiquement à des projets de synthèse vocale. C’est un excellent point de départ.|
|Œuvres libres<br>de droits|Il s’agit en général, d’œuvres publiées avant 1923. En anglais, le [projet Gutenberg](https://www.gutenberg.org/) propose des dizaines de milliers de ces œuvres. Vous pouvez vous concentrer sur des œuvres plus récentes car leur contenu sera plus proche de l’anglais moderne.|
|Œuvres&nbsp;du gouvernement|Les œuvres créées par l’état fédéral des États-Unis ne sont pas soumises au copyright, mais cela peut être le cas dans d’autres pays/régions.|
|Domaine public|Œuvres pour lesquelles tout copyright a été explicitement exclu, ou qui sont destinées au domaine public. Certaines juridictions interdisent tout renoncement total au copyright.|
|Œuvres cédées sous licence|Œuvres distribuées sous licence comme Creative Commons ou la Licence de documentation libre GNU (GFDL). Wikipédia utilise une licence GFDL. Certaines licences, toutefois, peuvent imposer des restrictions quant à une utilisation du contenu sous licence qui pourrait avoir un impact sur la création d’un modèle de voix personnalisée. Par conséquent, lisez attentivement la licence.|

## <a name="recording-your-script"></a>Enregistrement de votre script

Enregistrez votre script dans un studio d’enregistrement professionnel spécialisé dans le travail de la voix. Ce type de studio est équipé d’une cabine d’enregistrement, d’un équipement approprié et d’un personnel compétent formé à son utilisation. Veillez à ne pas négliger l’enregistrement.

Discutez de votre projet avec l’ingénieur du son chargé de l’enregistrement et écoutez ses conseils. L’enregistrement doit avoir peu voire aucune compression de plage dynamique (maximum 4:1). Il est essentiel que l’audio affiche un volume homogène et un rapport signal/bruit élevé, tout en étant exempt de sons parasites.

### <a name="do-it-yourself"></a>Faites-le vous-même

Si vous souhaitez effectuer vous-même l’enregistrement au lieu de faire appel à un studio d’enregistrement, voici quelques conseils pour bien démarrer. Avec la popularité croissante des enregistrements à domicile et des podcasts, il est plus facile que jamais de trouver des conseils et des ressources en ligne pour réussir ses enregistrements.

Pour votre cabine d’enregistrement, choisissez une petite pièce, sans écho notable ni « tonalité de pièce ». Elle doit être aussi silencieuse et insonorisée que possible. Vous pouvez fixer aux murs des rideaux afin de réduire l’écho et de neutraliser ou d’étouffer le son de la pièce.

Utilisez un microphone à condenseur studio de haute qualité (un « micro » pour faire plus court) adapté à l’enregistrement vocal. Les micros Sennheiser, AKG voire les derniers modèles Zoom produisent de bons résultats. Vous pouvez acheter un micro ou en louer un dans une entreprise de location de matériel audio-vidéo. Recherchez un modèle doté d’une interface USB. Ce type de micro combine de façon optimale un microphone, un préampli et un convertisseur analogique-numérique, ce qui simplifie la connexion.

Vous pouvez également utiliser un microphone analogique. De nombreuses entreprises de location proposent des microphones « vintage » réputés pour la qualité de la voix qu’ils reproduisent. Notez qu’un système analogique professionnel utilise des connecteurs XLR équilibrés plutôt que les prises d’un quart de pouce (1/4”) dont sont équipés les systèmes grand public. Si vous optez pour un système analogique, vous aurez également besoin d’un préampli et d’une interface audio pour ordinateur avec ces connecteurs.

Installez le microphone sur un support ou sur un pied, puis placez un filtre anti-pop devant le microphone pour supprimer le bruit des consonnes plosives comme « p » et « b ». Certains microphones sont munis d’un système de suspension qui les isole des vibrations du support, ce qui est utile.

L’acteur doit se tenir à une distance constante du microphone. Collez sur le sol des bandes pour lui indiquer la position idéale. S’il préfère travailler assis, surveillez tout particulièrement la distance par rapport au micro et évitez tout bruit de chaise.

Posez le script sur un pupitre. Évitez toute inclinaison de pupitre qui risquerait de réverbérer le son vers le microphone.

La personne qui commande l’équipement — l’ingénieur du son — doit se trouver dans un local distinct de la cabine d’enregistrement où se trouve l’acteur, et disposer d’un moyen de communiquer avec lui (*circuit d’ordre).*

L’enregistrement doit contenir le moins de bruits parasites possible, avec pour objectif un rapport signal/bruit de 80 dB ou moins.

Écoutez attentivement l’enregistrement d’un silence dans votre cabine et identifiez la source des éventuels bruits afin de les supprimer. Les sources courantes de bruit sont les orifices de ventilation, les ballasts de néons, la circulation environnante et les ventilateurs des équipements (même un petit notebook peut avoir des ventilateurs). Les microphones et les câbles peuvent générer un bruit électrique (un bourdonnement ou un sifflement) s’ils sont placés à proximité d’une climatisation. Un bourdonnement peut également être dû à une *boucle de masse*, présente quand un appareil est raccordé à plusieurs circuits électriques.

> [!TIP]
> Dans certains cas, vous pouvez utiliser un égaliseur ou un plug-in logiciel de réduction du bruit pour éliminer le bruit de vos enregistrements, même s’il est toujours préférable de le supprimer à sa source.

Réglez les niveaux afin d’utiliser au maximum la plage dynamique disponible de l’enregistrement numérique sans saturation. Cela signifie que vous devez régler le son à un niveau élevé, mais pas si élevé qu’il en devient déformé. L’image suivante montre un exemple de forme d’onde d’un bon enregistrement :

![Forme d’onde d’un bon enregistrement](media/custom-voice/good-recording.png)

Ici, la majeure partie de la plage (hauteur) est utilisée, mais les pics les plus élevés du signal n’atteignent pas le haut ou le bas de la fenêtre. Vous pouvez également constater que la latence dans l’enregistrement ressemble à une fine ligne horizontale, indiquant un faible niveau de bruit. Cet enregistrement comporte une plage dynamique et un rapport signal/bruit acceptables.

Enregistrez directement sur l’ordinateur par le biais d’une interface audio de haute qualité ou d’un port USB, en fonction du micro que vous utilisez. Pour un système analogique, gardez une chaîne audio simple : micro, préampli, interface audio, ordinateur. Vous pouvez acheter à un prix raisonnable une licence des logiciels [Avid Pro Tools](https://www.avid.com/en/pro-tools) et [Adobe Audition](https://www.adobe.com/products/audition.html). Si votre budget est extrêmement serré, essayez la version gratuite d’[Audacity](https://www.audacityteam.org/).

Effectuez un enregistrement de type 44,1 kHz 16 bits monophonique (qualité CD) ou plus. Si votre équipement le permet, utilisez un échantillonnage 48 kHz 24 bits pour obtenir un son exceptionnel. Vous réduirez l’échantillonnage à 16 kHz 16 bits avant de l’envoyer au portail Custom Voice. Mais il est préférable de disposer d’un enregistrement original de haute qualité, si des modifications s’avèrent nécessaires.

Dans l’idéal, différentes personnes occupent les rôles de directeur, ingénieur du son et acteur. N’essayez pas de tout faire vous-même. Faute de mieux, une même personne peut endosser le costume de directeur et d’ingénieur du son.

### <a name="before-the-session"></a>Avant la session

Pour éviter toute perdre de temps au studio, parcourez le script avec votre acteur avant la session d’enregistrement. À mesure qu’il se familiarise avec le texte, l’acteur apprend à prononcer correctement les éventuels mots inconnus.

> [!NOTE]
> La plupart des studios d’enregistrement sont équipés d’un écran électronique pour afficher les scripts dans la cabine d’enregistrement. Dans ce cas, saisissez vos notes directement dans le document du script. Mais conservez une copie papier pour prendre des notes pendant la session. Les ingénieurs du son demandent aussi souvent une copie papier. Et gardez toujours une troisième copie imprimée en secours pour l’acteur, au cas où l’ordinateur tombe en panne.

L’acteur vous demandera parfois de lui préciser le mot à mettre en relief dans un énoncé (« mot opérationnel »). Dites-lui que vous souhaitez une lecture naturelle, sans mise en relief particulière. Une mise en relief peut être ajoutée lorsque la reconnaissance vocale est synthétisée : elle ne doit pas faire partie de l’enregistrement d’origine.

Demandez à l’acteur de prononcer distinctement tous les mots. Chaque mot du script doit être prononcé. Aucun son ne doit être omis ou avalé, comme c’est souvent le cas dans une conversation informelle,  *sauf si le script a été écrit dans ce sens.*

|Texte écrit|Prononciation informelle indésirable|
|-|-|
|never going to give you up|never gonna give you up|
|there are four lights|there're four lights|
|how's the weather today|how's th' weather today|
|say hello to my little friend|say hello to my lil' friend|

L’acteur ne doit *pas* ajouter de pause entre chaque mot. La phrase doit toujours être prononcée de façon naturelle, même si elle sonne un peu formelle. La maîtrise de cette nuance peut demander du temps.

### <a name="the-recording-session"></a>La session d'enregistrement

Créez un enregistrement de référence, ou *fichier de correspondance,* d’un énoncé standard en début de session. Demandez à l’acteur de répéter cette ligne à chaque page environ. À chaque fois, comparez le nouvel enregistrement et la référence. Cet exercice aide l’acteur à conserver un niveau constant de volume, de tempo et d’intonation. Pendant ce temps, l’ingénieur du son peut utiliser le fichier de correspondance comme référence pour les niveaux et l’homogénéité globale du son.

Le fichier de correspondance est particulièrement important pour reprendre l’enregistrement après une pause ou à une date ultérieure. Vous pouvez jouer plusieurs fois ce fichier à l’acteur afin de le faire répéter, jusqu'à ce que les deux versions se ressemblent.

Demandez à l’acteur de respirer profondément et de faire une pause avant chaque énoncé. Enregistrez quelques secondes de silence entre les énoncés. Les mots doivent être prononcés de la même façon chaque fois qu’ils apparaissent, en prenant en compte le contexte.

Enregistrez cinq bonnes secondes de silence avant le premier enregistrement pour saisir la « tonalité de la pièce ». Cette pratique aide le portail Custom Voice à atténuer les éventuels bruits parasites restants.

> [!TIP]
> Comme vous avez avant tout besoin de la voix de l’acteur, vous pouvez effectuer un enregistrement monophonique (monocanal) de ces lignes. Mais s’il s’agit d’un enregistrement en stéréo, vous pouvez utiliser le second canal pour capturer le son provenant de la salle de contrôle et écouter ainsi les discussions concernant des lignes ou des prises particulières. Supprimez cette piste de la version chargée sur le portail Custom Voice.

Écoutez attentivement, à l’aide d’un casque, les enregistrements de la voix professionnelle. Vous devez obtenir une diction naturelle et une prononciation claire, sans bruits parasites. N’hésitez pas à demander à l’acteur de répéter un énoncé, si vous estimez que ces critères ne sont pas remplis.

> [!TIP]
> Si vous utilisez un grand nombre d’énoncés, un énoncé unique peut ne pas avoir d’effet notable sur la voix personnalisée finale. Il est parfois préférable de noter simplement les énoncés qui posent problème, puis de les exclure de votre jeu de données pour examiner le résultat final de la voix personnalisée. Vous pourrez toujours retourner en studio afin d’y enregistrer les échantillons manquants.

Pour chaque énoncé, notez sur le script le numéro de la prise ou le code temporel. Demandez également à l’ingénieur du son de marquer si possible chaque énoncé dans les métadonnées ou le « cue sheet » de l’enregistrement.

Faites régulièrement des pauses et proposez une boisson à l’acteur pour maintenir une bonne qualité de voix.

### <a name="after-the-session"></a>Après la session

Les studios d’enregistrement modernes travaillent sur ordinateur. À la fin de la session, vous recevez un ou plusieurs fichiers audio, pas une bande. Ces fichiers seront probablement au format WAV ou AIFF en qualité CD (44,1 kHz 16 bits) ou plus. Un échantillonnage 48 kHz 24 bits est courant et recommandé. Les taux d’échantillonnage plus élevés, par exemple 96 kHz, ne sont généralement pas nécessaires.

Le portail Custom Voice nécessite la création d’un fichier spécifique pour chaque énoncé. Chaque fichier audio fourni par le studio contient plusieurs énoncés. Par conséquent, la principale tâche de post-production consiste à diviser les enregistrements et à les préparer pour l’envoi. L’ingénieur du son de l’enregistrement peut avoir placé des marqueurs dans le fichier (ou créé un « cue sheet » distinct) pour indiquer où chaque énoncé commence.

Utilisez vos notes pour identifier exactement vos prises, puis un utilitaire de montage comme [Avid Pro Tools](https://www.avid.com/en/pro-tools), [Adobe Audition](https://www.adobe.com/products/audition.html) ou la version gratuite d’[Audacity](https://www.audacityteam.org/) pour copier chaque énoncé dans un nouveau fichier.

Laissez seulement environ 0,2 seconde de silence au début et à la fin de chaque clip, à l’exception du premier. Ce fichier doit commencer par cinq secondes de silence. N’utilisez pas d’éditeur audio pour supprimer les blancs du fichier. Ajoutez la « tonalité de la pièce » pour aider les algorithmes Custom Voice à atténuer les bruits de fond résiduels.

Écoutez attentivement chaque fichier. À ce stade, vous pouvez modifier les petits sons indésirables que vous avez manqués pendant l’enregistrement, par exemple un léger bruit de bouche avant une ligne, mais veillez à ne pas supprimer l’énoncé. Si vous ne pouvez pas corriger un fichier, supprimez-le de votre jeu de données et prenez note de cette opération.

Convertissez chaque fichier au format 16 bits et à un taux d’échantillonnage de 16 kHz avant l’enregistrement et, si vous avez enregistré les conversations du studio, supprimez le second canal. Enregistrez chaque fichier au format WAV, puis nommez les fichiers à l’aide des numéros d’énoncé de votre script.

Pour finir, créez la *transcription* qui associe chaque fichier WAV à une version texte de l’énoncé correspondant. La [création des polices de la voix personnalisée](./how-to-custom-voice-create-voice.md) fournit des informations sur le format requis. Vous pouvez copier le texte directement à partir de votre script. Créez ensuite un fichier Zip contenant les fichiers WAV et la transcription du texte.

Archivez les enregistrements originaux dans un endroit sûr afin de pouvoir les réutiliser ultérieurement si nécessaire. Conservez également votre script et vos notes.

## <a name="next-steps"></a>Étapes suivantes

Vous êtes prêt à charger vos enregistrements et à créer votre voix personnalisée.

> [!div class="nextstepaction"]
> [Créer des polices de la voix personnalisées](./how-to-custom-voice-create-voice.md)