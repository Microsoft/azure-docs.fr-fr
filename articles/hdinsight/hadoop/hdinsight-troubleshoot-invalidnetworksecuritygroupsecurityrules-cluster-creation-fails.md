---
title: Erreur InvalidNetworkSecurityGroupSecurityRules – Azure HDInsight
description: La création de cluster échoue en générant le code d’erreur InvalidNetworkSecurityGroupSecurityRules
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 07/31/2019
ms.openlocfilehash: b1423dc965a3169a5f615ccc371849cc177be244
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93289099"
---
# <a name="scenario-invalidnetworksecuritygroupsecurityrules---cluster-creation-fails-in-azure-hdinsight"></a>Scénario : InvalidNetworkSecurityGroupSecurityRules – Échec de la création d’un cluster dans Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.

## <a name="issue"></a>Problème

Vous recevez le code d’erreur `InvalidNetworkSecurityGroupSecurityRules`, accompagné d’une description similaire à ce qui suit : « Les règles de sécurité dans le Groupe de sécurité réseau configurées avec le sous-réseau n’autorisent pas la connectivité entrante et/ou sortante requise. »

## <a name="cause"></a>Cause

Ce problème est probablement lié aux règles du [groupe de sécurité réseau](../../virtual-network/virtual-network-vnet-plan-design-arm.md) entrant qui sont configurées pour votre cluster.

## <a name="resolution"></a>Résolution

Accédez au Portail Microsoft Azure et identifiez le groupe de sécurité réseau associé au sous-réseau dans lequel le cluster est en cours de déploiement. Dans la section **Règles de sécurité de trafic entrant** , assurez-vous que les règles autorisent l’accès entrant au port 443 pour les adresses IP indiquées [ici](../control-network-traffic.md).

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]