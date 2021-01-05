---
title: Erreur Jupyter 404 – « Blocage de l’API Cross Origin » – Azure HDInsight
description: Erreur 404 " introuvable " du serveur Jupyter en raison d’un " blocage d’API Cross Origin " dans Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 07/29/2019
ms.openlocfilehash: f2000c3938e7366dd85f36b8de9a8425e91fab8a
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93287940"
---
# <a name="scenario-jupyter-server-404-not-found-error-due-to-blocking-cross-origin-api-in-azure-hdinsight"></a>Scénario : Erreur 404 " introuvable " du serveur Jupyter en raison d’un " blocage d’API Cross Origin " dans Azure HDInsight

Cet article décrit la procédure à suivre pour résoudre les problèmes rencontrés lors de l’utilisation de composants Apache Spark dans des clusters Azure HDInsight.

## <a name="issue"></a>Problème

Lorsque vous accédez au service Jupyter sur HDInsight, un message d’erreur indiquant « Introuvable » s’affiche. Si vous consultez les journaux Jupyter, vous obtenez un résultat semblable à ce qui suit :

```log
[W 2018-08-21 17:43:33.352 NotebookApp] 404 PUT /api/contents/PySpark/notebook.ipynb (10.16.0.144) 4504.03ms referer=https://pnhr01hdi-corpdir.msappproxy.net/jupyter/notebooks/PySpark/notebook.ipynb
Blocking Cross Origin API request.  
Origin: https://xxx.xxx.xxx, Host: pnhr01.j101qxjrl4zebmhb0vmhg044xe.ax.internal.cloudapp.net:8001
```

Vous pouvez également voir une adresse IP dans le champ « Origine » dans le journal Jupyter.

## <a name="cause"></a>Cause

Cette erreur peut avoir les causes suivantes :

- Si vous avez configuré des règles de groupe de sécurité réseau (NSG) pour limiter l’accès au cluster. La restriction de l’accès avec des règles NSG vous permet toujours d’accéder directement à Apache Ambari et à d’autres services à l’aide de l’adresse IP plutôt que du nom de cluster. Toutefois, lors de l’accès à Jupyter, vous pouvez voir une erreur 404 « Introuvable ».

- Si vous avez donné à votre passerelle HDInsight un nom DNS personnalisé autre que le `xxx.azurehdinsight.net` standard.

## <a name="resolution"></a>Résolution

1. Modifiez les fichiers jupyter.py dans ces deux emplacements :

    ```bash
    /var/lib/ambari-server/resources/common-services/JUPYTER/1.0.0/package/scripts/jupyter.py
    /var/lib/ambari-agent/cache/common-services/JUPYTER/1.0.0/package/scripts/jupyter.py
    ```

1. Recherchez la ligne qui indique : `NotebookApp.allow_origin='\"https://{2}.{3}\"'`. Et remplacez-la par : `NotebookApp.allow_origin='\"*\"'`.

1. Redémarrez le service Jupyter à partir d’Ambari.

1. La saisie de `ps aux | grep jupyter` à l’invite de commandes doit indiquer qu’elle autorise toute URL s’y connecter.

Ce paramètre est moins sécurisé que celui que nous avions déjà mis en place. Mais l’accès au cluster est supposé restreint et un utilisateur extérieur est autorisé à se connecter au cluster car un groupe de sécurité réseau est en place.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]