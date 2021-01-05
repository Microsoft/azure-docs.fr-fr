---
title: Fichier include
description: Fichier include
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 01/14/2020
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: 65729934ea7c4037d6857aec10b14cdddd616368
ms.sourcegitcommit: 7e97ae405c1c6c8ac63850e1b88cf9c9c82372da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/29/2020
ms.locfileid: "97805625"
---
Les options de redondance pour un compte de stockage comprennent :

* Stockage localement redondant (LRS) : Une stratégie de redondance simple et économique. Les données sont copiées de façon synchrone trois fois au sein d’un même emplacement physique dans la région primaire.
* Stockage redondant interzone (ZRS) : Redondance pour les scénarios nécessitant une haute disponibilité. Les données sont copiées de façon synchrone sur trois zones de disponibilité Azure dans la région primaire.
* Stockage géoredondant (GRS) : Redondance entre les régions pour vous protéger en cas d’interruption régionale. Les données sont copiées de manière synchrone trois fois dans la région primaire, puis copiées de façon asynchrone dans la région secondaire. Pour un accès en lecture aux données dans la région secondaire, activez le stockage géoredondant avec accès en lecture (RA-GRS).
* Stockage géoredondant interzone (GZRS) (préversion) : Redondance pour les scénarios nécessitant à la fois une haute disponibilité et une durabilité maximale. Les données sont copiées de manière synchrone dans trois zones de disponibilité Azure dans la région primaire, puis copiées de façon asynchrone dans la région secondaire. Pour un accès en lecture aux données dans la région secondaire, activez le stockage géoredondant interzone avec accès en lecture (RA-GZRS).

Pour plus d’informations sur les options de redondance dans Stockage Azure, consultez [Redondance de Stockage Azure](../articles/storage/common/storage-redundancy.md).
