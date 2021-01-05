---
title: Fonctionnement du routage avec Azure Dev Spaces
services: azure-dev-spaces
ms.date: 03/24/2020
ms.topic: conceptual
description: Décrit les processus sur lesquels repose Azure Dev Spaces et le fonctionnement du routage
keywords: Azure Dev Spaces, Dev Spaces, Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, conteneurs
ms.openlocfilehash: 2d2c6f336222b4ae0907d6579289a8cad8d73aa6
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91977967"
---
# <a name="how-routing-works-with-azure-dev-spaces"></a>Fonctionnement du routage avec Azure Dev Spaces

[!INCLUDE [Azure Dev Spaces deprecation](../../includes/dev-spaces-deprecation.md)]

Azure Dev Spaces vous offre plusieurs façons de parcourir et de déboguer rapidement des applications Kubernetes et de collaborer avec votre équipe sur un cluster Azure Kubernetes Service (AKS). Une fois que votre projet s’exécute dans un espace de développement, Azure Dev Spaces fournit des fonctionnalités de routage et de mise en réseau supplémentaires pour votre projet.

Cet article décrit le fonctionnement du routage avec les espaces de développement.

## <a name="how-routing-works"></a>Fonctionnement du routage

Un espace de développement est créé par dessus AKS et utilise les mêmes [concepts de mise en réseau](../aks/concepts-network.md). Azure Dev Spaces dispose également d'un service *ingressmanager* centralisé et déploie son propre contrôleur d'entrée sur le cluster AKS. Le service *ingressmanager* surveille les clusters AKS avec des espaces de développement et augmente le contrôleur d'entrée Azure Dev Spaces dans le cluster avec des objets d’entrée pour le routage vers les pods d’application. Le conteneur devspaces-proxy de chaque pod ajoute un en-tête HTTP `azds-route-as` pour le trafic HTTP vers un espace de développement basé sur l'URL. Par exemple, une requête à l'URL *http://azureuser.s.default.serviceA.fedcba09...azds.io* obtiendrait un en-tête HTTP avec `azds-route-as: azureuser`. Le conteneur devspaces-proxy n'ajoutera pas d'en-tête `azds-route-as` s'il y en a déjà un.

Lorsqu'une requête HTTP est envoyée à un service extérieur au cluster, la requête est transmise au contrôleur d’entrée. Le contrôleur d’entrée achemine la requête directement au pod approprié en fonction de ses objets et règles d’entrée. Le conteneur devspaces-proxy du pod reçoit la requête, ajoute l'en-tête `azds-route-as` basé sur l'URL, puis achemine la requête vers le conteneur d’application.

Lorsqu'une requête HTTP est envoyée à un service depuis un autre service au sein du cluster, la requête transite d'abord par le conteneur devspaces-proxy du service appelant. Le conteneur devspaces-proxy examine la requête HTTP et vérifie l'en-tête `azds-route-as`. Selon l'en-tête, le conteneur devspaces-proxy cherchera l'adresse IP du service associé à la valeur de cet en-tête. Si une adresse IP est trouvée, le conteneur devspaces-proxy redirige la requête vers cette adresse IP. Si aucune adresse IP n'est trouvée, le conteneur devspaces-proxy achemine la requête vers le conteneur de l’application parente.

Par exemple, les applications *serviceA* et *serviceB* sont déployées sur un espace de développement parent appelé *default* (par défaut). *serviceA* s'appuie sur *serviceB* et lui passe des appels HTTP. Azure User crée un espace de développement enfant basé sur l'espace *default* (par défaut), appelé *azureuser*. Azure User déploie également sa propre version de *serviceA* sur son espace enfant. Lorsqu'une requête est envoyée à *http://azureuser.s.default.serviceA.fedcba09...azds.io* :

![Routage Azure Dev Spaces](media/how-dev-spaces-works/routing.svg)

1. Le contrôleur d’entrée recherche l'adresse IP du pod associé à l'URL, soit *serviceA.azureuser*.
1. Le contrôleur d'entrée trouve l'adresse IP du pod dans l'espace de développement de l'utilisateur Azure et achemine la requête vers le pod *serviceA.azureuser*.
1. Le conteneur devspaces-proxy du pod *serviceA.azureuser* reçoit la requête et ajoute `azds-route-as: azureuser` comme en-tête HTTP.
1. Le conteneur devspaces-proxy du pod *serviceA.azureuser* achemine la requête vers le conteneur *serviceA* dans le pod *serviceA.azureuser*.
1. L'application *serviceA* du pod *serviceA.azureuser* appelle *serviceB*. L'application *serviceA* contient également un code pour conserver l'en-tête `azds-route-as` existant, soit `azds-route-as: azureuser` dans ce cas.
1. Le conteneur devspaces-proxy du pod *serviceA.azureuser* reçoit la requête et recherche l'IP de *serviceB* selon la valeur de l'en-tête `azds-route-as`.
1. Le conteneur devspaces-proxy du pod *serviceA.azureuser* ne trouve pas d'IP pour *serviceB.azureuser*.
1. Le conteneur devspaces-proxy du pod *serviceA.azureuser* cherche l'IP pour *serviceB* dans l'espace parent, soit *serviceB.default*.
1. Le conteneur devspaces-proxy du pod *serviceA.azureuser* trouve l'IP pour *serviceB.default* et achemine la requête vers le pod *serviceB.default*.
1. Le conteneur devspaces-proxy du pod *serviceB.default* reçoit la requête et l'achemine vers le conteneur d’application *serviceB* dans le pod *serviceB.default*.
1. L'application *serviceB* du pod *serviceB.default* renvoie une réponse au pod *serviceA.azureuser*.
1. Le conteneur devspaces-proxy du pod *serviceA.azureuser* reçoit la réponse et l’achemine vers le conteneur d’application *serviceA* dans le pod *serviceA.azureuser*.
1. L'application *serviceA* reçoit la réponse et renvoie ensuite sa propre réponse.
1. Le conteneur devspaces-proxy du pod *serviceA.azureuser* reçoit la réponse du conteneur d'application *serviceA* et achemine la réponse vers l'appelant original en dehors du cluster.

Tout autre trafic TCP non HTTP transite par le contrôleur d’entrée et les conteneurs devspaces-proxy ne sont pas modifiés.

## <a name="sharing-a-dev-space"></a>Partage d’un espace de développement

Lorsque vous travaillez en équipe, vous pouvez partager un espace de développement avec tous les membres de cette équipe et créer des espaces de développement dérivés. Un espace de développement peut être utilisé par toute personne ayant un accès contributeur au groupe de ressources de cet espace de développement.

Vous pouvez également créer un espace de développement dérivé d'un autre espace de développement. Lorsque vous créez un espace de développement dérivé, l’étiquette *azds.io/parent-space=PARENT-SPACE-NAME* est ajouté à l'espace de noms de l'espace de développement dérivé. De plus, toutes les applications de l'espace de développement parent sont partagées avec l'espace de développement dérivé. Si vous déployez une version mise à jour d'une application dans l'espace de développement dérivé, elle n'existera que dans cet espace de développement dérivé, et l'espace de développement parent ne sera pas affecté. Vous pouvez avoir un maximum de trois niveaux d'espaces de développement dérivés ou espaces *grands-parents*.

L'espace de développement dérivé acheminera également de façon intelligente les requêtes entre ses propres applications et les applications partagées par son parent. Le routage fonctionne en tentant d'acheminer la requête vers une application dans l'espace de développement dérivé, puis en revenant à l'application partagée depuis l'espace de développement parent. Le routage reviendra à l'application partagée dans l'espace grand-parent si l'application ne figure pas dans l'espace parent.

Par exemple :
* L’espace de développement *default* possède des applications *serviceA* et *serviceB*.
* L’espace de développement *azureuser* est dérivé de *default*.
* Une version mise à jour de *serviceA* est déployée sur *azureuser*.

Lorsque vous utilisez *azureuser*, toutes les requêtes envoyées à *serviceA* seront acheminées vers la version mise à jour dans *azureuser*. Une requête envoyée à *serviceB* tente d’abord d’être acheminée vers la version *azureuser* de *serviceB*. Comme elle n’existe pas, elle sera acheminée vers la version *default* de *serviceB*. Si la version *azureuser* de *serviceA* est supprimée, toutes les requêtes envoyées à *serviceA* seront retournées à l’aide de la version *default* de *serviceA*.

## <a name="next-steps"></a>Étapes suivantes

Pour voir un exemple de la façon dont Azure Dev Spaces utilise le routage pour fournir une itération et un développement rapides, consultez [Comment fonctionne le débogage à distance de votre code avec Azure Dev Spaces][how-it-works-remote-debugging].


[how-it-works-remote-debugging]: how-dev-spaces-works-remote-debugging.md