---
title: hbase hbck retourne des incohérences dans Azure HDInsight
description: hbase hbck retourne des incohérences dans Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 08/08/2019
ms.openlocfilehash: a3f6a709d3e0f1c71f89473b6ed8435c6eea7e1c
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92540225"
---
# <a name="scenario-hbase-hbck-command-returns-inconsistencies-in-azure-hdinsight"></a>Scénario : la commande `hbase hbck` retourne des incohérences dans Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.

## <a name="issue-region-is-not-in-hbasemeta"></a>Problème : La région n’est pas dans `hbase:meta`

Région xxx sur HDFS, mais non listée dans `hbase:meta` ou déployée sur un serveur de région.

### <a name="cause"></a>Cause

Varie.

### <a name="resolution"></a>Résolution

1. Corrigez la table méta en exécutant :

    ```
    hbase hbck -ignorePreCheckPermission –fixMeta
    ```

1. Attribuez des régions à RegionServers en exécutant :

    ```
    hbase hbck -ignorePreCheckPermission –fixAssignment
    ```
---

## <a name="issue-region-is-offline"></a>Problème : La région est hors connexion

Région XXX non déployée sur un RegionServer. Cela signifie que la région est dans `hbase:meta`, mais hors connexion.

### <a name="cause"></a>Cause

Varie.

### <a name="resolution"></a>Résolution

Mettez les régions en ligne en exécutant :

```
hbase hbck -ignorePreCheckPermission –fixAssignment
```

---

## <a name="issue-regions-have-the-same-startend-keys"></a>Problème : Les régions ont les mêmes clés de début/fin

### <a name="cause"></a>Cause

Varie.

### <a name="resolution"></a>Résolution

Fusionnez manuellement les régions qui se chevauchent. Accédez à la section de table de l’interface utilisateur web HBase HMaster, puis sélectionnez le lien de table qui présente le problème. Vous verrez la clé de début/clé de fin de chaque région appartenant à cette table. Fusionnez ensuite les régions qui se chevauchent. Dans l’interpréteur de commandes HBase, effectuez `merge_region 'xxxxxxxx','yyyyyyy', true`. Par exemple :

```
RegionA, startkey:001, endkey:010,

RegionB, startkey:001, endkey:080,

RegionC, startkey:010, endkey:080.
```

Dans ce scénario, vous devez fusionner RegionA et RegionC, définir RegionD avec la même plage de clés que RegionB, puis fusionner RegionB et RegionD. xxxxxxx et yyyyyy correspondent à la chaîne de hachage à la fin de chaque nom de région. Veillez à ne pas fusionner deux régions discontinues. Après chaque fusion, comme la fusion de A et C, HBase démarre un compactage sur RegionD. Attendez la fin du compactage avant d’effectuer une autre fusion avec RegionD. Vous pouvez trouver l’état du compactage sur la page de ce serveur de région dans l’interface utilisateur HBase HMaster.

---

## <a name="issue-cant-load-regioninfo"></a>Problème : Impossible de charger `.regioninfo`

Impossible de charger `.regioninfo` pour la région `/hbase/data/default/tablex/regiony`.

### <a name="cause"></a>Cause

Cela est probablement dû à la suppression partielle de la région lorsque RegionServer se bloque ou la machine virtuelle redémarre. Actuellement, le Stockage Azure est un système de fichiers d’objets blob plat et certaines opérations sur les fichiers ne sont pas atomiques.

### <a name="resolution"></a>Résolution

Nettoyez manuellement ces fichiers et dossiers restants :

1. Exécutez `hdfs dfs -ls /hbase/data/default/tablex/regiony` pour vérifier quels dossiers/fichiers sont toujours en dessous.

1. Exécutez `hdfs dfs -rmr /hbase/data/default/tablex/regiony/filez` pour supprimer tous les fichiers/dossiers enfants

1. Exécutez `hdfs dfs -rmr /hbase/data/default/tablex/regiony` pour supprimer le dossier de région.

---

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas dans cet article ou si vous ne parvenez pas à le résoudre, utilisez un des canaux suivants pour obtenir de l’aide :

* Obtenez des réponses de la part d’experts Azure en faisant appel au [Support de la communauté Azure](https://azure.microsoft.com/support/community/).

* Connectez-vous à [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client. Connexion de la communauté Azure aux ressources appropriées : réponses, support technique et experts.

* Si vous avez besoin d’une aide supplémentaire, vous pouvez envoyer une requête de support à partir du [Portail Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Sélectionnez **Support** dans la barre de menus, ou ouvrez le hub **Aide + Support** . Pour plus d’informations, consultez [Création d’une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). L’accès au support relatif à la gestion et à la facturation des abonnements est inclus avec votre abonnement Microsoft Azure. En outre, le support technique est fourni avec l’un des [plans de support Azure](https://azure.microsoft.com/support/plans/).