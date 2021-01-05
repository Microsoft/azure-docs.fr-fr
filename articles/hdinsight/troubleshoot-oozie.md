---
title: Résoudre les problèmes liés à Apache Oozie dans Azure HDInsight
description: Résolvez certaines erreurs Apache Oozie dans Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 04/27/2020
ms.openlocfilehash: edbe5274de8576fccb29e1e69d260a6531d4ab05
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93287393"
---
# <a name="troubleshoot-apache-oozie-in-azure-hdinsight"></a>Résoudre les problèmes liés à Apache Oozie dans Azure HDInsight

Avec l’interface utilisateur Apache Oozie, vous pouvez afficher les journaux Oozie. L’interface utilisateur Oozie contient également des liens vers les journaux d’activité JobTracker pour les tâches MapReduce démarrées par le workflow. Le modèle pour la résolution des problèmes doit être le suivant :

1. Affichez le travail dans l’interface utilisateur web Oozie.

2. En cas d’erreur ou d’échec d’une action spécifique, sélectionnez-la pour voir si le champ **Message d’erreur** donne plus d’informations sur l’échec.

3. Si elle est disponible, utilisez l’URL de l’action pour afficher des détails supplémentaires (tels que les journaux d’activité JobTracker) pour l’action.

Voici des erreurs spécifiques que vous pouvez rencontrer avec une description de la marche à suivre pour les résoudre.

## <a name="ja009-cant-initialize-cluster"></a>JA009 : Impossible d’initialiser le cluster

### <a name="issue"></a>Problème

l’état de la tâche passe à **SUSPENDED** (Suspendu). Les détails du travail montrent que l’état de `RunHiveScript` est défini sur **START_MANUAL**. Lorsque vous sélectionnez l’action, le message d’erreur suivant apparaît :

```output
JA009: Cannot initialize Cluster. Please check your configuration for map
```

### <a name="cause"></a>Cause

les adresses de Stockage Blob Azure utilisées dans le fichier **job.xml** ne contiennent pas le conteneur de stockage ou le nom du compte de stockage. Le format d’adresse de stockage blob doit être `wasbs://containername@storageaccountname.blob.core.windows.net`.

### <a name="resolution"></a>Résolution

changez les adresses de Stockage Blob utilisées par la tâche.

---

## <a name="ja002-oozie-isnt-allowed-to-impersonate-ltusergt"></a>JA002 : Oozie ne peut pas emprunter l’identité &lt;USER&gt;

### <a name="issue"></a>Problème

l’état de la tâche passe à **SUSPENDED** (Suspendu). Les détails du travail montrent que l’état de `RunHiveScript` est défini sur **START_MANUAL**. Lorsque vous sélectionnez l’action, le message d’erreur suivant s’affiche :

```output
JA002: User: oozie is not allowed to impersonate <USER>
```

### <a name="cause"></a>Cause

les paramètres d’autorisation actuels ne permettent pas à Oozie d’emprunter l’identité du compte d’utilisateur spécifié.

### <a name="resolution"></a>Résolution

Oozie peut emprunter l’identité des utilisateurs du groupe **`users`** . Utilisez le `groups USERNAME` pour voir les groupes dont le compte d’utilisateur est membre. Si l’utilisateur n’est pas membre du groupe **`users`** , utilisez la commande suivante pour l’ajouter au groupe :

```bash
sudo adduser USERNAME users
```

> [!NOTE]  
> Il peut se passer plusieurs minutes avant que HDInsight reconnaisse que l’utilisateur a été ajouté au groupe.

---

## <a name="launcher-error-sqoop"></a>Launcher ERROR (Sqoop) (Erreur du lanceur, Sqoop)

### <a name="issue"></a>Problème

l’état de la tâche passe à **KILLED** (Supprimé). Les détails du travail montrent que l’état de `RunSqoopExport` est défini sur **ERROR**. Lorsque vous sélectionnez l’action, le message d’erreur suivant s’affiche :

```output
Launcher ERROR, reason: Main class [org.apache.oozie.action.hadoop.SqoopMain], exit code [1]
```

### <a name="cause"></a>Cause

Sqoop ne peut pas charger le pilote de base de données nécessaire pour accéder à la base de données.

### <a name="resolution"></a>Résolution

quand vous utilisez Sqoop à partir d’une tâche Oozie, vous devez inclure le pilote de base de données avec les autres ressources (comme workflow.xml) utilisées par la tâche. Vous devez également référencer l’archive contenant le pilote de base de données à partir de la section `<sqoop>...</sqoop>` de workflow.xml.

Ainsi, pour l’exemple de travail dans [Utiliser des workflows Hadoop Oozie](hdinsight-use-oozie-linux-mac.md), vous utilisez les étapes suivantes :

1. Copiez le fichier `mssql-jdbc-7.0.0.jre8.jar` dans le répertoire **/tutorials/useoozie** :

    ```bash
    hdfs dfs -put /usr/share/java/sqljdbc_7.0/enu/mssql-jdbc-7.0.0.jre8.jar /tutorials/useoozie/mssql-jdbc-7.0.0.jre8.jar
    ```

2. Modifiez le fichier `workflow.xml` pour ajouter le code XML suivant sur une nouvelle ligne au-dessus de `</sqoop>` :

    ```xml
    <archive>mssql-jdbc-7.0.0.jre8.jar</archive>
    ```

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../includes/hdinsight-troubleshooting-next-steps.md)]