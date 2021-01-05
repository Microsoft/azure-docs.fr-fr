---
title: Erreurs de nom de ressource réservé
description: Explique comment résoudre les erreurs quand vous fournissez un nom de ressource qui inclut un mot réservé.
ms.topic: troubleshooting
ms.date: 11/08/2017
ms.openlocfilehash: e76f4bf9bfee7de6e7523d69acf1388d2dd80e93
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75474259"
---
# <a name="resolve-reserved-resource-name-errors"></a>Résoudre les erreurs de nom de ressource réservé

Cet article décrit l’erreur que vous rencontrez quand vous déployez une ressource dont le nom inclut un mot réservé.

## <a name="symptom"></a>Symptôme

Au cours du déploiement d’une ressource qui est disponible par le biais d’un point de terminaison public, vous pouvez rencontrer l’erreur suivante :

```
Code=ReservedResourceName;
Message=The resource name <resource-name> or a part of the name is a trademarked or reserved word.
```

## <a name="cause"></a>Cause

Le nom d’une ressource qui a un point de terminaison public ne peut pas contenir de mots réservés ou de marques.

Les mots suivants sont réservés :

* ACCESS
* AZURE
* BING
* BIZSPARK
* BIZTALK
* CORTANA
* DIRECTX
* DOTNET
* DYNAMICS
* EXCEL
* EXCHANGE
* FOREFRONT
* GROOVE
* HOLOLENS
* HYPERV
* KINECT
* LYNC
* MSDN
* O365
* OFFICE
* OFFICE365
* ONEDRIVE
* ONENOTE
* OUTLOOK
* POWERPOINT
* SHAREPOINT
* SKYPE
* VISIO
* VISUALSTUDIO

Les mots suivants ne peuvent pas être utilisés en tant que mot entier ou sous-chaîne dans le nom :

* Connexion
* MICROSOFT
* WINDOWS
* XBOX

## <a name="solution"></a>Solution

Fournissez un nom qui n’utilise aucun des mots réservés.