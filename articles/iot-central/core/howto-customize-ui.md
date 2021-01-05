---
title: Personnaliser l’interface utilisateur Azure IoT Central | Microsoft Docs
description: Comment personnaliser le thème et les liens d’aide pour votre application Azure IoT Central
author: dominicbetts
ms.author: dobett
ms.date: 12/06/2019
ms.topic: how-to
ms.service: iot-central
services: iot-central
manager: philmea
ms.openlocfilehash: 660f6dfae23331455408ae7127274550a3a4cc3a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90019833"
---
# <a name="customize-the-azure-iot-central-ui"></a>Personnaliser l’interface utilisateur d’Azure IoT Central

Cet article explique comment, en tant qu’administrateur, vous pouvez personnaliser l’interface utilisateur de votre application en appliquant des thèmes personnalisés et en modifiant les liens d’aide pour pointer vers vos propres ressources d’aide personnalisées. 

La capture d’écran suivante montre une page utilisant le thème standard :

![Thème standard de IoT Central](./media/howto-customize-ui/standard-ui.png)

La capture d’écran suivante montre une page dont les éléments personnalisés ont été encadrés :

![Thème IoT Central personnalisé](./media/howto-customize-ui/themed-ui.png)

## <a name="create-theme"></a>Créer un thème

Pour créer un thème personnalisé, dans la section **Administration**, accédez à la page **Personnaliser votre application** :

![Thèmes de IoT Central](./media/howto-customize-ui/themes.png)

Dans cette page, vous pouvez personnaliser les aspects suivants de votre application :

### <a name="application-logo"></a>Logo de l’application

Une image PNG, ne dépassant pas 1 Mo, avec un fond transparent. Ce logo s’affiche à gauche sur la barre de titre de l’application IoT Central.

Si l’image de votre logo inclut le nom de votre application, vous pouvez masquer le texte du nom de votre application. Pour en savoir plus, veuillez consulter la section [Gérer votre application](howto-administer.md#change-application-name-and-url).

### <a name="browser-icon-favicon"></a>Icône de navigateur (favicon)

Une image PNG, ne dépassant pas 32 x 32 pixels, avec un fond transparent. Un navigateur web peut utiliser cette image dans la barre d’adresse, l’historique, les signets et l’onglet du navigateur.

### <a name="browser-colors"></a>Couleurs du navigateur

Vous pouvez modifier la couleur de l’en-tête de la page et la couleur utilisée pour accentuer les boutons et les mises en surbrillance. Utilisez une valeur de couleur hexadécimale de six caractères au format `##ff6347`. Pour plus d’informations sur la notation des couleurs en **valeur hexadécimale (HEX)** , veuillez consulter la page [Couleurs HTML](https://www.w3schools.com/html/html_colors.asp).

> [!NOTE]
> Vous pouvez toujours revenir aux options par défaut sur la page **Personnaliser votre application**.

### <a name="changes-for-operators"></a>Modifications pour les opérateurs

Si un administrateur crée un thème personnalisé, les opérateurs et autres utilisateurs de votre application ne pourront plus choisir un thème dans **Paramètres**.

## <a name="replace-help-links"></a>Remplacer les liens d’aide

Pour fournir des informations d’aide personnalisées à vos opérateurs et autres utilisateurs, vous pouvez modifier les liens dans le menu **Aide** de l’application.

Pour modifier les liens d’aide, dans la section **Administration**, accédez à la page **Personnaliser l’aide** :

![Personnaliser les liens d’aide d’IoT Central](./media/howto-customize-ui/help-links.png)

Vous pouvez également ajouter de nouvelles entrées au menu d’aide et supprimer les entrées par défaut :

![Aide personnalisée d’IoT Central](./media/howto-customize-ui/custom-help.png)

> [!NOTE]
> Vous pouvez toujours revenir aux liens d’aide par défaut sur la page **Personnaliser l’aide**.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez découvert comment personnaliser l’interface utilisateur de votre application Azure IoT Central, voici quelques suggestions d’étapes suivantes :

- [Administrer votre application](./howto-administer.md)
- [Ajouter des vignettes à votre tableau de bord](howto-add-tiles-to-your-dashboard.md)