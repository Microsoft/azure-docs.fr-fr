---
title: Instructions pour les expériences d’ancrage efficaces
description: Instructions et considérations pour une création et une localisation efficaces des ancres à l’aide d’Azure Spatial Anchors.
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 11/20/2020
ms.topic: conceptual
ms.service: azure-spatial-anchors
ms.openlocfilehash: a6bc500516de7e554c38a335ea57519a39c8f602
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95487501"
---
# <a name="create-an-effective-anchor-experience-by-using-azure-spatial-anchors"></a>Créer une expérience d’ancrage efficace à l’aide d’Azure Spatial Anchors

Cet article donne des instructions et soulève des considérations pour vous aider à créer et localiser des ancres de manière efficace avec Azure Spatial Anchors.

## <a name="anchor-improvement-over-time"></a>Amélioration de l’ancre au fil du temps

Avec Azure Spatial Anchors, chaque fois que vous localisez des ancres, nous tentons d’améliorer la qualité des opérations de localisation futures. Pour ce faire, nous utilisons les données d’environnement collectées pour compléter les informations visuelles sur les ancres que nous recherchons. Ce processus s’exécute en arrière-plan et est une optimisation hors connexion exécutée par le service Azure Spatial Anchors pour optimiser votre environnement. Les données supplémentaires collectées au cours de chaque opération permettent de mieux comprendre l’environnement. Vous pouvez ainsi améliorer la qualité et la localisation des ancres grâce à la modification de l’environnement, au passage du temps et aux utilisateurs qui regardent les ancres à partir de différents angles et perspectives.

## <a name="good-anchors"></a>Des ancres de qualité

Même si Azure Spatial Anchors tente d’améliorer la qualité des ancres au fil du temps, il est également important de consacrer du temps à la formation ou au guidage des utilisateurs dans votre expérience utilisateur (UX) afin de créer de bons points d’ancrage. Un tel investissement en amont vous permet d’aider les utilisateurs à trouver des ancres fiables :

- Sur différents appareils.
- À des moments différents.
- Dans des conditions d’éclairage différentes.
- Selon les perspectives souhaitées dans l’espace.

## <a name="static-and-dynamic-locations"></a>Localisations statiques et dynamiques

Une partie de la conception de l’expérience d’ancrage consiste à choisir les localisations. Les localisations seront-elles statiques et définies par un administrateur de l’espace ? Ou seront-elles dynamiques et définies par l’utilisateur ?

Un gérant de magasin de vente au détail pourra avoir besoin d’une expérience statique en magasin pour inciter les utilisateurs à venir sur place. En revanche, le développeur d’un jeu de plateau de réalité mixte préférera probablement permettre aux utilisateurs de choisir eux-mêmes où jouer.

Pour les localisations statiques, vous pouvez apprendre aux administrateurs à passer du temps à organiser l’espace avec des ancres de qualité.

Pour les localisations dynamiques, réfléchissez à la manière de former ou guider les utilisateurs dans votre expérience utilisateur pour créer des ancres de qualité.

## <a name="stable-visual-features"></a>Des caractéristiques visuelles stables

Les systèmes de suivi visuel utilisés sur des appareils de réalité mixte et de réalité augmentée s’appuient sur les caractéristiques visuelles de l’environnement. Pour obtenir l’expérience la plus fiable :

- *Créez* des ancres sur des localisations dont les caractéristiques visuelles sont stables (autrement dit, des caractéristiques qui ne changent pas souvent).

- *Ne créez pas* d’ancres sur de grandes surfaces vierges dépourvues de caractéristiques distinctives.

- *Ne créez pas* d’ancres sur des matériaux très réfléchissants.

- *Ne créez pas* d’ancres sur des surfaces où un motif se répète, par exemple un tapis ou du papier peint.

![Exemples de bon environnement et de mauvais environnement pour les ancres](./media/stable-visual.png)

## <a name="various-viewing-perspectives"></a>Différentes perspectives de visualisation

Quand vous créez une ancre, pensez aux personnes qui vont essayer de la localiser par la suite.

Prenons l’exemple d’une ancre située au milieu d’une salle qui a deux portes. Vous voulez probablement que les utilisateurs puissent entrer dans la salle par l’une ou l’autre des deux portes. Quand vous créez l’ancre, vous avez besoin de scanner sa position à partir des deux portes. Vous changez de perspectives pour capturer des données d’environnement autour de l’ancre afin que les utilisateurs puissent la localiser depuis l’une ou l’autre des portes.

En règle générale, quand vous créez une ancre, scannez-la selon les perspectives des personnes qui vont essayer de la localiser. Ainsi, si vous placez du contenu virtuel sur une sculpture en plein air, il est judicieux, pour créer l’ancre, d’en faire tout le tour pendant le scan. Si votre ancre se trouve dans le coin d’une salle, il n’y a alors qu’une seule direction d’approche. Pour créer cette ancre, vous pouvez tout simplement la scanner selon cette perspective.

## <a name="multiple-anchors"></a>Multiplicité des ancres

L’éclairage peut faire la différence dans les caractéristiques visuelles qu’une application détecte. Les ancres créées sous une lumière naturelle forte peuvent être difficiles à localiser sous une lumière artificielle, et vice versa.

Si vous rencontrez ce problème, la création de deux ancres peut s’avérer salutaire. Au même endroit, créez une première ancre sous la lumière du jour, puis une autre sous une lumière artificielle. Votre application peut alors rechercher les deux ancres. Quand l’une ou l’autre est localisée, l’application sait où la poser.

De même, dans les environnements où les caractéristiques visuelles changent en raison du déplacement de la plupart des objets, il est judicieux d’utiliser plusieurs ancres. Quand une ancre devient trop difficile à trouver en raison d’importants changements dans l’environnement, vous pouvez la remplacer par une autre. Ce peut être le cas, par exemple, dans un magasin de vente au détail où l’agencement est retravaillé à chaque changement de saison.

## <a name="targets-and-rooms"></a>Cibles et salles

Dans de nombreux cas, une ancre est un point d’entrée dans l’expérience de votre application. Cette étape doit être rapide et fiable pour que les utilisateurs puissent accéder facilement à votre expérience. Passer du temps sur la façon dont les utilisateurs vont trouver vos ancres est une étape de conception incontournable. Il convient de réfléchir à la recherche des ancres selon deux scénarios généraux : les *cibles* et les *salles*.

### <a name="targets"></a>Cibles

Dans le scénario avec des cibles, la localisation d’une ancre est connue. Par exemple, dans une application de peinture fictive en réalité mixte, un utilisateur place une toile sur le mur. Il demande aux autres utilisateurs présents dans la salle de pointer leurs appareils vers ce même endroit sur le mur pour localiser l’ancre afin de démarrer l’expérience.

Autre exemple de scénario avec des cibles : une pancarte dans une brûlerie indiquant « Scannez pour découvrir les offres ». La brûlerie a placé une ancre à cet endroit. Quand les clients scannent la pancarte, ils localisent l’ancre et entrent dans l’expérience de réalité augmentée pour découvrir les offres spéciales sur le café.

Les photos peuvent s’avérer une aide précieuse dans le scénario avec des cibles. Si vous montrez aux utilisateurs une photo de la cible à trouver sur leur appareil, ils peuvent rapidement identifier ce qu’ils doivent scanner dans le monde réel. Par exemple, vous pouvez guider vos utilisateurs vers la zone large d’une cible précise par GPS. Quand l’utilisateur y parvient, votre application montre une photo de la cible. L’utilisateur regarde autour de lui, trouve la cible et recherche l’ancre à scanner.

![Illustration d’une ancre, montrant une photo de la cible sur l’appareil mobile d’un utilisateur](./media/start-here-edit.png)

### <a name="rooms"></a>Salles

Dans le scénario avec des salles, les utilisateurs pénètrent dans un espace en sachant simplement qu’il s’y trouve une ancre quelque part. Ils scannent l’espace avec leur appareil pour repérer rapidement l’ancre.

Cette expérience vous oblige généralement à créer des ancres bien organisées, comme mentionné dans la section Différentes perspectives de visualisation. Si vous avez scanné la salle selon de nombreuses perspectives quand vous avez créé l’ancre, les utilisateurs peuvent scanner presque n’importe où quand ils essaient de la localiser.

![Illustration de la façon dont un utilisateur peut scanner une salle pour trouver une ancre](./media/scan-room.png)

Au fond, vous consacrez plus de temps à scanner l’espace quand vous créez l’ancre pour que les utilisateurs puissent scanner l’espace et localiser l’ancre rapidement par la suite. Au moment où vous créez votre expérience, vous devrez envisager ce compromis important.

L’exemple d’application de peinture en réalité mixte dont nous avons parlé plus tôt ne fonctionne pas bien dans le scénario avec des salles. Ici, l’utilisateur qui place l’ancre veut que d’autres personnes rejoignent l’expérience rapidement. Les utilisateurs n’ont pas envie d’attendre de démarrer l’expérience tant qu’ils n’ont pas scanné la salle dans sa totalité. Puisque tous les utilisateurs savent exactement vers où pointer leur appareil pour localiser les ancres, cet exemple fonctionne mieux comme scénario avec des cibles.

## <a name="anchor-location"></a>Localisation d’une ancre

Les systèmes de suivi visuel s’appuient sur les caractéristiques visuelles d’un environnement. Plus un scan comporte de caractéristiques visuelles, plus les chances sont élevées de trouver une ancre.

Suivez les recommandations générales données dans cette section pour créer une expérience utilisateur qui encourage un scan utile de l’environnement.

Tout d’abord, si l’utilisateur ne localise pas une ancre en quelques secondes, l’application doit l’encourager à déplacer l’appareil pour capturer d’autres perspectives. L’application peut aussi encourager les utilisateurs à se déplacer dans l’environnement pour rechercher l’ancre depuis d’autres perspectives. Plus l’appareil perçoit des perspectives de fonctionnalités, mieux c’est, car cela augmente la probabilité qu’une ancre soit localisée, ainsi que le recueil d’autres données sur l’environnement, qui seront utilisées pour améliorer la qualité de l’ancre.

Pour les scénarios avec des cibles, demandez à l’utilisateur de se déplacer autour de la cible pour la voir depuis des perspectives différentes. En d’autres termes, demandez à l’utilisateur de capturer la cible depuis des perspectives nouvelles jusqu’à ce qu’il trouve l’ancre.

Pour les scénarios avec des salles, demandez-lui de scanner lentement la pièce. Par exemple, demandez à l’utilisateur de tourner sur lui-même pour capturer 180 degrés voire 360 degrés de la salle. Ou demandez-lui de voir la salle depuis une nouvelle perspective.

La méthode la plus significative consiste à traverser la salle pendant le scan. Un scan en traversant la pièce permet de capturer plus de caractéristiques visuelles de l’environnement qu’un scan à proximité d’un mur, par exemple. Un scan à proximité d’un mur ne capture pas autant de caractéristiques visuelles utiles de l’environnement.

Il ne sert à rien non plus de déplacer l’appareil de gauche à droite plusieurs fois pour rechercher une ancre. Les mêmes points depuis de la même perspective sont alors tout simplement capturés.

## <a name="experience-tests"></a>Tests de l’expérience

Dans cet article, nous avons vu les recommandations générales. Avec Spatial Anchors, vous écrivez des applications qui interagissent avec le monde réel. C’est pour cela que vous devez consacrer du temps à tester les scénarios d’ancrage de votre application dans des environnements réels. Cela est d’autant plus vrai pour les environnements qui correspondent aux endroits où vous espérez que vos utilisateurs se servent de votre application.
