---
title: Configurer le transfert DNS pour Azure Red Hat OpenShift 4
description: Configurer le transfert DNS pour Azure Red Hat OpenShift 4
author: sakthi-vetrivel
ms.author: suvetriv
ms.service: container-service
ms.topic: conceptual
ms.date: 04/24/2020
ms.openlocfilehash: 6784c69c049945f3bd4977ecd647d57ec59bf9f1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82232630"
---
# <a name="configure-dns-forwarding-on-an-azure-red-hat-openshift-4-cluster"></a>Configurer le transfert DNS sur un cluster Azure Red Hat OpenShift 4

Pour configurer le transfert DNS sur un cluster Azure Red Hat OpenShift, vous devez modifier l’opérateur DNS. Cette modification permet à vos pods d’application de s’exécuter au sein du cluster pour résoudre les noms hébergés sur un serveur DNS privé en dehors du cluster. Ces étapes sont documentées pour OpenShift 4.3 [ici](https://docs.openshift.com/container-platform/4.3/networking/dns-operator.html).

Par exemple, si vous souhaitez transférer toutes les demandes DNS pour que *.example.com soit résolu par un serveur DNS 192.168.100.10, vous pouvez modifier la configuration de l’opérateur en exécutant la commande suivante :
 
```bash
oc edit dns.operator/default
```
 
Dans l’éditeur qui s’ouvre, vous pouvez remplacer `spec: {}` par :
 
 ```yaml
spec:
  servers:
  - forwardPlugin:
      upstreams:
      - 192.168.100.10
    name: example-dns
    zones:
    - example.com
```

Enregistrez le fichier, puis quittez l’éditeur.

## <a name="next-steps"></a>Étapes suivantes
Vous trouverez [ici](https://docs.openshift.com/container-platform/4.3/networking/dns-operator.html) plus d’informations sur le transfert DNS pour OpenShift 4.3.