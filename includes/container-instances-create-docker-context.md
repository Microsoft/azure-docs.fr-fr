---
title: Fichier include
description: Fichier include
services: container-instances
author: dlepow
ms.service: container-instances
ms.topic: include
ms.date: 08/13/2020
ms.author: danlep
ms.custom: include file
ms.openlocfilehash: 3221c4f3e196cf1573bd7c0424fa3b4530c0b2ca
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90708250"
---
## <a name="create-azure-context"></a>Créer un contexte Azure

Pour utiliser les commandes Docker pour exécuter des conteneurs dans Azure Container Instances, connectez-vous d’abord à Azure :

```bash
docker login azure
```

À l’invite, entrez ou sélectionnez vos informations d’identification Azure.


Créez un contexte ACI en exécutant `docker context create aci`. Ce contexte associe Docker à un abonnement Azure et un groupe de ressources, ce qui vous permet de créer et gérer des instances de conteneur. Par exemple, pour créer un contexte appelé *myacicontext* :

```
docker context create aci myacicontext
```

Lorsque vous y êtes invité, sélectionnez votre ID d’abonnement Azure, puis sélectionnez un groupe de ressources existant ou **créez-en-un**. Si vous choisissez un nouveau groupe de ressources, celui-ci est créé avec un nom généré par le système. Les instances de conteneur Azure, comme toutes les ressources Azure, doivent être déployées dans un groupe de ressources. Les groupes de ressources vous permettent d’organiser et de gérer les ressources Azure connexes.


Exécutez `docker context ls` pour confirmer que vous avez ajouté le contexte ACI à vos contextes Docker :

```
docker context ls
```