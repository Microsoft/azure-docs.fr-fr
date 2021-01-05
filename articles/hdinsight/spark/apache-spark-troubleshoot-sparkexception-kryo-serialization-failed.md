---
title: Problèmes avec JDBC/ODBC et le framework Apache Thrift – Azure HDInsight
description: Impossible de télécharger de grands jeux de données à l’aide de JDBC/ODBC et de l’infrastructure logicielle Apache Thrift dans Azure HDInsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 05/14/2020
ms.openlocfilehash: aa97d5caf481502cf6dc6e0aa26742c26d47a02e
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92538984"
---
# <a name="unable-to-download-large-data-sets-using-jdbcodbc-and-apache-thrift-software-framework-in-hdinsight"></a>Impossible de télécharger de grands jeux de données à l’aide de JDBC/ODBC et de l’infrastructure logicielle Apache Thrift dans HDInsight

Cet article décrit la procédure à suivre pour résoudre les problèmes rencontrés lors de l’utilisation de composants Apache Spark dans des clusters Azure HDInsight.

## <a name="issue"></a>Problème

Lorsque vous essayez de télécharger de grands jeux de données à l’aide de JDBC/ODBC et de l’infrastructure logicielle Apache Thrift dans Azure HDInsight, un message d'erreur similaire au suivant s'affiche :

```
org.apache.spark.SparkException: Kryo serialization failed:
Buffer overflow. Available: 0, required: 36518. To avoid this, increase spark.kryoserializer.buffer.max value.
```

## <a name="cause"></a>Cause

Cette exception est liée au processus de sérialisation qui tente d’utiliser plus d’espace de mémoire tampon qu'autorisé. Dans Spark 2.0.0, la classe `org.apache.spark.serializer.KryoSerializer` est utilisée pour sérialiser des objets en cas d'accès aux données via l’infrastructure logicielle Apache Thrift. Une autre classe est utilisée pour les données envoyées via le réseau ou mises en cache sous forme sérialisée.

## <a name="resolution"></a>Résolution

Augmentez valeur de la mémoire tampon `Kryoserializer`. Ajoutez une clé nommée `spark.kryoserializer.buffer.max` et définissez-la sur `2047` dans la configuration spark2 sous `Custom spark2-thrift-sparkconf`. Redémarrez tous les composants concernés.

> [!IMPORTANT]
> Cette valeur de `spark.kryoserializer.buffer.max` doit être inférieure à 2048. Les valeurs fractionnaires ne sont pas prises en charge.

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas dans cet article ou si vous ne parvenez pas à le résoudre, utilisez un des canaux suivants pour obtenir de l’aide :

* Obtenez des réponses de la part d’experts Azure en faisant appel au [Support de la communauté Azure](https://azure.microsoft.com/support/community/).

* Connectez-vous avec [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client en connectant la communauté Azure aux ressources appropriées (réponses, support et experts).

* Si vous avez besoin d’une aide supplémentaire, vous pouvez envoyer une requête de support à partir du [Portail Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Sélectionnez **Support** dans la barre de menus, ou ouvrez le hub **Aide + Support** . Pour plus d’informations, consultez [Création d’une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). L’accès au support relatif à la gestion et à la facturation des abonnements est inclus avec votre abonnement Microsoft Azure. En outre, le support technique est fourni avec l’un des [plans de support Azure](https://azure.microsoft.com/support/plans/).