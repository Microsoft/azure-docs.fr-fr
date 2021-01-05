---
title: Activer l’affinité basée sur les cookies avec Application Gateway
description: Cet article fournit des informations sur la façon d’activer l’affinité basée sur les cookies à l’aide d’une Application Gateway.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/4/2019
ms.author: caya
ms.openlocfilehash: a7806cf9518090539ba540a9a522af1aae2691f0
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397414"
---
# <a name="enable-cookie-based-affinity-with-an-application-gateway"></a>Activer l’affinité basée sur les cookies à l’aide d’une Application Gateway
Comme indiqué dans la [documentation d’Azure Application Gateway](./application-gateway-components.md#http-settings), Application Gateway prend en charge l’affinité basée sur les cookies, ce qui signifie que le service peut diriger le trafic qui en résulte à partir d’une session utilisateur vers le même serveur à des fins de traitement.

## <a name="example"></a>Exemple
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: guestbook
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
    appgw.ingress.kubernetes.io/cookie-based-affinity: "true"
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: frontend
          servicePort: 80
```