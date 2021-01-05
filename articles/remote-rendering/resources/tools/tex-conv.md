---
title: TexConv - Outil de conversion de texture
description: Liens vers le dépôt d’outils de texture sur GitHub
author: jakrams
ms.author: jakras
ms.date: 02/11/2020
ms.topic: article
ms.openlocfilehash: 2ad1645e55dcaa9a5b6461a9972c2922d5e32ebc
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94693203"
---
# <a name="texconv---texture-conversion-tool"></a>TexConv - Outil de conversion de texture

L’outil TexConv est un outil en ligne de commande externe, qui permet d’effectuer des conversions de texture par lots.
L’outil se trouve dans ce dépôt Git : [Dépôt GitHub pour l’outil TexConv](https://github.com/microsoft/DirectXTex/wiki/Texconv).

Pour Remote Rendering, les textures n’ont pas besoin d’être converties, car l’étape de conversion permet de traiter les formats de fichiers image courants et de convertir les textures au format de runtime approprié. Toutefois, cet outil peut être utile pour convertir des textures localement avec des paramètres personnalisés.
