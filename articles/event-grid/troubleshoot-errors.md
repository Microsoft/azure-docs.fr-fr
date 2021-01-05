---
title: Azure Event Grid - Guide de résolution des problèmes
description: Cet article fournit une liste de codes d’erreur avec les messages d’erreur correspondants, leur description et les actions recommandées.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 79533918ccc6995f459b39f058de9e01091c0958
ms.sourcegitcommit: 1cf157f9a57850739adef72219e79d76ed89e264
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94592989"
---
# <a name="troubleshoot-azure-event-grid-errors"></a>Résoudre les erreurs liées à Azure Event Grid
Ce guide de résolution des problèmes fournit les informations suivantes : 

- Codes d’erreur Azure Event Grid
- Messages d’erreur
- Descriptions des erreurs
- Actions recommandées à exécuter lorsque ces erreurs se produisent. 

## <a name="error-code-400"></a>Code d’erreur : 400
| Code d'erreur | Message d’erreur | Description | Recommandation |
| ---------- | ------------- | ----------- | -------------- | 
| HttpStatusCode.BadRequest<br/>400 | Le nom de la rubrique doit compter 3 à 50 caractères. | La longueur du nom de la rubrique personnalisée doit être comprise entre 3 et 50 caractères. Seuls les caractères alphanumériques, les chiffres et le caractère « - » sont autorisés dans le nom de la rubrique. De plus, le nom ne doit pas commencer par les mots réservés suivants : <ul><li>Microsoft-</li><li>EventGrid-</li><li>System-</li></ul> | Choisissez un autre nom conforme aux exigences relatives aux noms de rubrique. |
| HttpStatusCode.BadRequest<br/>400 | Le nom de domaine doit compter 3 à 50 caractères. | La longueur du nom de domaine doit être comprise entre 3 et 50 caractères. Seuls les caractères alphanumériques, les chiffres et le caractère « - » sont autorisés dans le nom de domaine. De plus, le nom ne doit pas commencer par les mots réservés suivants :<ul><li>Microsoft-</li><li>EventGrid-</li><li>System-</li> | Choisissez un autre nom conforme aux exigences relatives aux noms de domaine. |
| HttpStatusCode.BadRequest<br/>400 | Délai d’expiration non valide. | Le délai d’expiration de l’abonnement aux événements détermine à quel moment l’abonnement aux événements sera mis hors service. Cette valeur doit être une valeur DateHeure valide dans le futur.| Assurez-vous que le délai d’expiration de l’abonnement aux événements respecte un format DateHeure valide et qu’il est défini dans le futur. |

## <a name="error-code-409"></a>Code d’erreur : 409
| Code d'erreur | Message d’erreur | Description | Action recommandée |
| ---------- | ------------- | ----------- | -------------- | 
| HttpStatusCode.Conflict <br/>409 | Une rubrique existe déjà sous le nom spécifié. Choisissez un autre nom de rubrique.   | Le nom de la rubrique personnalisée doit être unique au sein d’une même région Azure pour que l’opération de publication s’effectue correctement. Il est possible d’utiliser le même nom dans des régions Azure différentes. | Choisissez un autre nom de rubrique. |
| HttpStatusCode.Conflict <br/> 409 | Un domaine existe déjà sous le nom spécifié. Choisissez un autre nom de domaine. | Le nom de domaine doit être unique au sein d’une même région Azure pour que l’opération de publication s’effectue correctement. Il est possible d’utiliser le même nom dans des régions Azure différentes. | Choisissez un autre nom de domaine. |
| HttpStatusCode.Conflict<br/>409 | Limite du quota atteinte. Pour plus d’informations sur ces limites, consultez [Limites d’Azure Event Grid](../azure-resource-manager/management/azure-subscription-service-limits.md#event-grid-limits).  | Le nombre de ressources Azure Event Grid que peut utiliser chaque abonnement Azure est limité. Tout ou partie de ce quota a été dépassé et aucune ressource supplémentaire n’a pu être créée. |    Vérifiez l’utilisation des ressources actuelles et supprimez celles qui ne sont pas nécessaires. Si vous avez quand même besoin d’augmenter votre quota, envoyez un e-mail à [aeg@microsoft.com](mailto:aeg@microsoft.com) en indiquant le nombre exact de ressources nécessaires. |

## <a name="error-code-403"></a>Code d’erreur : 403

| Code d'erreur | Message d’erreur | Description | Action recommandée |
| ---------- | ------------- | ----------- | ------------------ |
| HttpStatusCode.Forbidden <br/>403 | La publication sur {Rubrique/Domaine} par le client {AdresseIP} est rejetée en raison des règles de filtrage d’adresse IP. | La rubrique ou le domaine a des règles de pare-feu IP configurées et l’accès est limité uniquement aux adresses IP configurées. | Ajoutez l’adresse IP aux règles de pare-feu IP ; consultez [Configurer le pare-feu IP](configure-firewall.md) |
| HttpStatusCode.Forbidden <br/> 403 | La publication sur {Rubrique/Domaine} par le client est rejetée, car la demande émane du point de terminaison privé et aucune connexion de point de terminaison privée correspondante n’a été trouvée pour la ressource. | La rubrique ou le domaine a des points de terminaison privés configurés et la demande de publication provient d’un point de terminaison privé qui n’est pas configuré ou approuvé. | Configurez un point de terminaison privé pour la rubrique/le domaine. [Configurer des points de terminaison privés](configure-private-endpoints.md) |

Vérifiez également qu'il se trouve derrière une instance d'Azure Application Gateway ou un pare-feu d'applications web. Si c’est le cas, désactivez les règles de pare-feu suivantes et exécutez une nouvelle requête HTTP POST :

- 920300 (En-tête Accept manquant dans la requête)
- 942430 (Détection restreinte des anomalies de caractères SQL (args) : nombre de caractères spéciaux dépassé (12))
- 920230 (Détection d'encodage de plusieurs URL)
- 942130 (Attaque par injection de code SQL : Tautologie SQL détectée.)
- 931130 (Attaque possible par inclusion de fichier distant = Référence/Lien hors domaine)



## <a name="next-steps"></a>Étapes suivantes
Si vous avez besoin d’une aide supplémentaire, publiez votre problème sur le forum [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-eventgrid) ou ouvrez un [ticket de support](https://azure.microsoft.com/support/options/). 
