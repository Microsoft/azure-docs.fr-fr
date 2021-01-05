---
title: Créer des pages d’erreur personnalisées pour Azure Application Gateway
description: Cet article explique comment créer des pages d’erreur personnalisées pour Application Gateway. Vous pouvez utiliser votre marque et votre mise en page personnelle à l’aide d’une page d’erreur personnalisée.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/16/2019
ms.author: victorh
ms.openlocfilehash: 5b34c559c8320961a2e96a663d88001400c572d3
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397516"
---
# <a name="create-application-gateway-custom-error-pages"></a>Créer des pages d’erreur personnalisées pour Application Gateway

Application Gateway vous permet de créer des pages d’erreur personnalisées au lieu d’afficher les pages d’erreur par défaut. Vous pouvez utiliser votre marque et votre mise en page personnelle à l’aide d’une page d’erreur personnalisée.

Par exemple, vous pouvez définir votre propre page de maintenance si votre application web n’est pas accessible. Ou bien, vous pouvez créer une page d’accès non autorisé si une requête malveillante est envoyée à une application web.

Les pages d’erreur personnalisées sont prises en charge dans les deux scénarios suivants :

- **Page maintenance** : cette page d’erreur personnalisée est envoyée en remplacement d’une page 502 signalant une passerelle incorrecte. Elle est affichée lorsque Application Gateway ne dispose d’aucun serveur principal vers lequel acheminer le trafic. C’est par exemple le cas lorsqu’une maintenance est planifiée ou lorsqu’un problème imprévu a des répercussions sur l’accès au pool principal.
- **Page d’accès non autorisé** : cette page d’erreur personnalisée est envoyée en remplacement d’une page 403 signalant un accès non autorisé. Elle s’affiche lorsque le pare-feu d’applications web pour la passerelle Application Gateway détecte du trafic malveillant et le bloque.

Si une erreur provient des serveurs principaux, elle est alors retransmise à l’appelant, sans modification. Aucune page d’erreur personnalisée ne s’affiche. Application Gateway peut afficher une page d’erreur personnalisée lorsqu’une requête ne peut pas atteindre le serveur principal.

Rien n’empêche de définir des pages d’erreur personnalisées au niveau global et de l’écouteur :

- **Niveau global** : la page d’erreur concerne le trafic de toutes les applications web déployées sur la passerelle Application Gateway.
- **Niveau écouteur** : la page d’erreur concerne le trafic reçu sur cet écouteur.
- **Les deux** : la page d’erreur personnalisée définie au niveau de l’écouteur remplace celle définie au niveau global.

Pour créer une page d’erreur personnalisée, vous devez disposer des éléments suivants :

- le code d’état d’une réponse HTTP
- l’emplacement correspondant de la page d’erreur 
- un blob de Stockage Azure accessible publiquement pour cet emplacement
- une extension de type *.htm ou *.html 

La taille de la page d’erreur doit être inférieure à 1 Mo. Si des images sont liées dans la page d’erreur, il doit s’agir d’images incluses dans la page d’erreur personnalisée, soit sous forme d’URL absolues accessibles publiquement, soit sous forme d’images codées en base64. Les liens relatifs contenant des images situées dans le même emplacement blob ne sont actuellement pas pris en charge. 

Après avoir spécifié une page d’erreur, la passerelle d’application la télécharge à partir de l’emplacement d’objets blob de stockage et l’enregistre dans le cache local de la passerelle d’application. Par la suite, la page d’erreur est prise en charge directement à partir de la passerelle d’application. Pour modifier une page d’erreur personnalisée existante, vous devez pointer vers un autre emplacement d’objet blob dans la configuration de la passerelle d’application. La passerelle d’application ne vérifie pas régulièrement l’emplacement des objets blob pour extraire les nouvelles versions.

## <a name="portal-configuration"></a>Configuration du portail

1. Accédez à la passerelle Application Gateway dans le portail et choisissez une passerelle d’application.

    ![Capture d'écran représentant la page de présentation d'une passerelle applicative.](media/custom-error/ag-overview.png)
2. Cliquez sur **Écouteurs** et accédez à un écouteur spécifique sur lequel vous souhaitez spécifier une page d’erreur.

    ![Écouteurs Application Gateway](media/custom-error/ag-listener.png)
3. Configurez une page d’erreur personnalisée pour une erreur de pare-feu d’applications web 403 ou une page de maintenance 502 au niveau de l’écouteur.

    > [!NOTE]
    > Actuellement, la création de pages d’erreur personnalisées au niveau global à partir du portail Microsoft Azure n’est pas prise en charge.

4. Spécifiez une URL d’objet blob accessible publiquement pour un code d’état d’erreur donné, puis cliquez sur **Enregistrer**. La passerelle Application Gateway est maintenant configurée avec la page d’erreur personnalisée.

   ![Codes d’erreur Application Gateway](media/custom-error/ag-error-codes.png)

## <a name="azure-powershell-configuration"></a>Configuration d’Azure PowerShell

Vous pouvez utiliser Azure PowerShell pour configurer une page d'erreur personnalisée. Par exemple, une page d'erreur personnalisée globale :

```powershell
$appgw   = Get-AzApplicationGateway -Name <app-gateway-name> -ResourceGroupName <resource-group-name>

$updatedgateway = Add-AzApplicationGatewayCustomError -ApplicationGateway $appgw -StatusCode HttpStatus502 -CustomErrorPageUrl "http://<website-url>"
```

Ou une page d'erreur au niveau d'un port d'écoute :

```powershell
$appgw   = Get-AzApplicationGateway -Name <app-gateway-name> -ResourceGroupName <resource-group-name>

$listener01 = Get-AzApplicationGatewayHttpListener -Name <listener-name> -ApplicationGateway $appgw

$updatedlistener = Add-AzApplicationGatewayHttpListenerCustomError -HttpListener $listener01 -StatusCode HttpStatus502 -CustomErrorPageUrl "http://<website-url>"
```

Pour plus d'informations, consultez [Add-AzApplicationGatewayCustomError](/powershell/module/az.network/add-azapplicationgatewaycustomerror?view=azps-1.2.0) et [Add-AzApplicationGatewayHttpListenerCustomError](/powershell/module/az.network/add-azapplicationgatewayhttplistenercustomerror?view=azps-1.3.0).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les diagnostics Application Gateway, voir [Intégrité du serveur principal, journaux de diagnostic et métriques pour la passerelle Application Gateway](application-gateway-diagnostics.md).