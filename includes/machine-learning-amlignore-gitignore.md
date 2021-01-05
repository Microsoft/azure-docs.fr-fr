---
author: blackmist
ms.service: machine-learning
ms.topic: include
ms.date: 05/06/2020
ms.author: larryfr
ms.openlocfilehash: 6a76e9be47827aee92d55df2679cc162bbd2414c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82871479"
---
Pour empêcher que les fichiers inutiles soient inclus dans l’instantané, créez un fichier « ignore» (`.gitignore` ou `.amlignore`) dans le répertoire. Ajoutez dans ce fichier les fichiers et répertoires à exclure. Pour plus d’informations sur la syntaxe à utiliser dans ce fichier, consultez [syntaxe et modèles](https://git-scm.com/docs/gitignore) pour `.gitignore`. Le fichier `.amlignore` utilise la même syntaxe. _Si les deux fichiers existent, le fichier `.amlignore` est prioritaire._
