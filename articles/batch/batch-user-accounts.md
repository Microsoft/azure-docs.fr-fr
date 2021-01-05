---
title: Exécuter des tâches sous des comptes d’utilisateur
description: Découvrez les types de comptes d’utilisateur et comment les configurer.
ms.topic: how-to
ms.date: 08/20/2020
ms.custom: seodec18
ms.openlocfilehash: cce374e7d7ffb513bed882b048ea54bcbad81b0b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88719357"
---
# <a name="run-tasks-under-user-accounts-in-batch"></a>Exécuter des tâches sous des comptes d’utilisateur dans Azure Batch

> [!NOTE]
> Pour des raisons de sécurité, les comptes d’utilisateur décrits dans cet article sont différents des comptes d’utilisateur utilisés pour le protocole RDP (Remote Desktop Protocol) ou le protocole SSH (Secure Shell).
>
> Pour vous connecter à un nœud qui exécute une configuration de machine virtuelle Linux via le protocole SSH, consultez [Installer et configurer le Bureau à distance pour effectuer une connexion à une machine virtuelle Linux dans Azure](../virtual-machines/linux/use-remote-desktop.md). Pour vous connecter à des nœuds exécutant Windows via RDP, consultez [Connexion à une machine virtuelle Azure exécutant Windows](../virtual-machines/windows/connect-logon.md).<br /><br />
> Pour vous connecter à un nœud qui exécute la configuration du service cloud via RDP, consultez [Activer une connexion Bureau à distance pour un rôle dans Azure Cloud Services](../cloud-services/cloud-services-role-enable-remote-desktop-new-portal.md).

Dans Azure Batch, une tâche s’exécute toujours sous un compte d’utilisateur. Par défaut, les tâches s’exécutent sous des comptes d’utilisateur standard qui ne possèdent pas de droits d’administrateur. Pour certains scénarios, vous pouvez configurer le compte d’utilisateur sous lequel vous voulez exécuter une tâche. Cet article décrit les types de comptes d’utilisateur et la manière de les configurer pour votre scénario.

## <a name="types-of-user-accounts"></a>Types de comptes d’utilisateur

Azure Batch offre deux types comptes d’utilisateur pour l’exécution des tâches :

- **Comptes d’utilisateur automatique.** Les comptes d’utilisateur automatique sont des comptes d’utilisateurs intégrés, créés automatiquement par le service Batch. Par défaut, les tâches sont exécutées sous un compte d’utilisateur automatique. Vous pouvez configurer la spécification d’utilisateur automatique pour une tâche afin d’indiquer le compte d’utilisateur automatique sous lequel la tâche doit être exécutée. La spécification d’utilisateur automatique vous permet de spécifier le niveau d’élévation et l’étendue du compte d’utilisateur automatique qui exécutera la tâche.

- **Un compte d’utilisateur nommé.** Vous pouvez spécifier un ou plusieurs comptes d’utilisateur nommé pour un pool au moment de sa création. Chaque compte d’utilisateur est créé sur chaque nœud du pool. En plus du nom de compte, vous pouvez spécifier le mot de passe utilisateur, le niveau d’élévation, ainsi que, pour les pools Linux, la clé privée SSH. Lorsque vous ajoutez une tâche, vous pouvez spécifier le compte d’utilisateur nommé sous lequel la tâche doit s’exécuter.

> [!IMPORTANT]
> La version du service Batch 2017-01-01.4.0 introduit une modification qui vous oblige à mettre à jour votre code pour appeler cette version. Si vous migrez du code à partir d’une version antérieure de Batch, notez que la propriété **runElevated** n’est plus prise en charge dans les bibliothèques d’API REST ou du client Batch. Utilisez la nouvelle propriété **userIdentity** d’une tâche pour spécifier le niveau d’élévation. Consultez [Mettre à jour votre code vers la dernière bibliothèque du client Batch](#update-your-code-to-the-latest-batch-client-library) pour savoir comment mettre à jour votre code Batch si vous utilisez l’une des bibliothèques clientes.

## <a name="user-account-access-to-files-and-directories"></a>Accès du compte d’utilisateur aux fichiers et répertoires

Un compte d’utilisateur automatique et un compte d’utilisateur nommé disposent tous deux d’un accès en lecture/écriture au répertoire de travail de la tâche, au répertoire partagé et au répertoire de tâches de plusieurs instances. Les deux types de comptes ont accès en lecture aux répertoires des tâches de démarrage et préparation d’un travail.

Si une tâche s’exécute sous le même compte que celui utilisé pour exécuter une tâche de démarrage, la tâche dispose d’un accès en lecture-écriture au répertoire de la tâche de démarrage. De la même manière, si une tâche s’exécute sous le même compte que celui utilisé pour exécuter une tâche de préparation d’un travail, la tâche dispose d’un accès en lecture-écriture au répertoire de la tâche de préparation. Si une tâche s’exécute sous un compte différent de celui utilisé pour la tâche de démarrage ou la tâche de préparation, la tâche dispose uniquement d’un accès en lecture au répertoire correspondant.

Pour plus d’informations sur l’accès aux fichiers et aux répertoires à partir d’une tâche, consultez la rubrique [Fichiers et répertoires](files-and-directories.md).

## <a name="elevated-access-for-tasks"></a>Accès aux tâches avec élévation de privilèges

Le niveau d’élévation du compte d’utilisateur indique si une tâche s’exécute dans le cadre d’un accès avec élévation de privilèges. Les comptes d’utilisateur automatique et nommé peuvent s’exécuter dans le cadre d’un accès avec élévation de privilèges. Il existe deux options de niveau d’élévation :

- **NonAdmin :** la tâche s’exécute en tant qu’utilisateur standard sans accès avec élévation de privilèges. Le niveau d’élévation par défaut pour un compte d’utilisateur Batch est toujours **NonAdmin**.
- **Admin :** la tâche s’exécute en tant qu’utilisateur disposant d’un accès avec élévation de privilèges et fonctionne avec des autorisations d’administrateur complètes.

## <a name="auto-user-accounts"></a>Comptes d’utilisateur automatique

Par défaut, les tâches s’exécutent dans Batch sous un compte d’utilisateur automatique, en tant qu’utilisateur standard qui ne dispose pas d’un accès avec élévation de privilèges, et avec une étendue de pool bien définie. L’étendue de pool signifie que la tâche s’exécute sous un compte d’utilisateur automatique disponible pour n’importe quelle tâche du pool. Pour plus d’informations sur l’étendue de pool, consultez [Exécution d’une tâche en tant qu’utilisateur automatique avec une étendue de pool](#run-a-task-as-an-auto-user-with-pool-scope).

L’étendue de pool peut être remplacée par une étendue de tâche. Lorsque la spécification d’utilisateur automatique est configurée pour l’étendue de la tâche, le service Batch crée un compte d’utilisateur automatique pour cette tâche uniquement.

Il existe quatre configurations possibles pour la spécification d’utilisateur automatique, chacune correspondant à un compte d’utilisateur automatique unique :

- Accès non-admin avec étendue de pool
- Accès admin (avec élévation de privilèges) avec étendue de la tâche
- Accès non-admin avec étendue de pool
- Accès admin avec étendue de pool

> [!IMPORTANT]
> Les tâches en cours d’exécution sous l’étendue de la tâche ne peuvent pas accéder aux tâches sur un nœud. Toutefois, un utilisateur malveillant ayant accès au compte pourrait contourner cette restriction en soumettant une tâche qui s’exécute avec des privilèges d’administrateur et qui accède à d’autres répertoires de la tâche. Un utilisateur malveillant pourrait également utiliser le protocole RDP ou SSH pour se connecter à un nœud. Il est important de protéger l’accès aux clés de votre compte Batch afin d’éviter un tel scénario. Si vous pensez que votre compte a peut-être été compromis, veillez à régénérer vos clés.

### <a name="run-a-task-as-an-auto-user-with-elevated-access"></a>Exécution d’une tâche en tant qu’utilisateur automatique disposant d’un accès avec élévation de privilèges

Vous pouvez configurer la spécification d’utilisateur automatique pour des privilèges d’administrateur lorsque vous avez besoin d’exécuter une tâche avec un accès avec élévation de privilèges. Par exemple, une tâche de démarrage peut avoir besoin d’un accès avec élévation de privilèges pour pouvoir installer un logiciel sur le nœud.

> [!NOTE]
> Utilisez l’accès avec élévation de privilèges uniquement lorsque cela est nécessaire. Les meilleures pratiques recommandent d’accorder le privilège minimal nécessaire pour atteindre le résultat souhaité. Par exemple, si une tâche de démarrage installe le logiciel pour l’utilisateur actuel, et non pour l’ensemble des utilisateurs, vous pouvez sans doute éviter d’accorder aux tâches un accès avec élévation de privilèges. Vous pouvez configurer la spécification d’utilisateur automatique d’étendue du pool et d’accès non-admin pour toutes les tâches qui doivent s’exécuter sous le même compte, y compris la tâche de démarrage.

Les extraits de code suivants montrent comment configurer la spécification d’utilisateur automatique. Les exemples définissent le niveau d’élévation sur `Admin` et l’étendue sur `Task`.

#### <a name="batch-net"></a>.NET Batch

```csharp
task.UserIdentity = new UserIdentity(new AutoUserSpecification(elevationLevel: ElevationLevel.Admin, scope: AutoUserScope.Task));
```
#### <a name="batch-java"></a>Java Batch

```java
taskToAdd.withId(taskId)
        .withUserIdentity(new UserIdentity()
            .withAutoUser(new AutoUserSpecification()
                .withElevationLevel(ElevationLevel.ADMIN))
                .withScope(AutoUserScope.TASK));
        .withCommandLine("cmd /c echo hello");
```

#### <a name="batch-python"></a>Python Batch

```python
user = batchmodels.UserIdentity(
    auto_user=batchmodels.AutoUserSpecification(
        elevation_level=batchmodels.ElevationLevel.admin,
        scope=batchmodels.AutoUserScope.task))
task = batchmodels.TaskAddParameter(
    id='task_1',
    command_line='cmd /c "echo hello world"',
    user_identity=user)
batch_client.task.add(job_id=jobid, task=task)
```

### <a name="run-a-task-as-an-auto-user-with-pool-scope"></a>Exécution d’une tâche en tant qu’utilisateur automatique avec une étendue de pool

Lorsqu’un nœud est approvisionné, deux comptes d’utilisateur automatique au niveau du pool sont créés sur chaque nœud du pool, l’un disposant d’un accès avec élévation de privilèges, l’autre non. En définissant l’étendue du compte d’utilisateur automatique sur l’étendue du pool pour une tâche donnée, la tâche s’exécute sous l’un de ces deux comptes d’utilisateur automatique au niveau du pool.

Lorsque vous spécifiez l’étendue du pool pour l’utilisateur automatique, toutes les tâches qui s’exécutent avec un accès administrateur s’exécutent sous le même compte d’utilisateur automatique au niveau du pool. De même, les tâches qui s’exécutent sans autorisations d’administrateur s’exécutent également sous un seul compte d’utilisateur automatique au niveau du pool.

> [!NOTE]
> Les deux comptes d’utilisateur automatique au niveau du pool sont des comptes distincts. Les tâches qui s’exécutent sous le compte d’administrateur au niveau du pool ne peuvent pas partager de données avec les tâches qui s’exécutent sous le compte standard, et inversement.

L’avantage d’exécuter une tâche sous le même compte d’utilisateur automatique est que les tâches sont en mesure de partager des données avec d’autres tâches en cours d’exécution sur le même nœud.

L’exécution de tâches sous l’un des deux comptes d’utilisateur automatique au niveau du pool est utile, par exemple, pour le partage de clés secrètes. Par exemple, supposons qu’une tâche de démarrage doive configurer une clé secrète dans un nœud utilisé par d’autres tâches. Vous pouvez utiliser l’API Windows Data Protection (DPAPI), mais elle requiert des privilèges d’administrateur. Au lieu de cela, vous pouvez protéger la clé secrète au niveau de l’utilisateur. Les tâches qui s’exécutent sous le même compte d’utilisateur peuvent accéder à la clé secrète sans accès avec élévation de privilèges.

Il peut être également intéressant d’exécuter des tâches sous un compte d’utilisateur automatique avec une étendue de pool dans le cas d’un partage de fichiers MPI (Message Passing Interface). Un partage de fichiers MPI est utile lorsque les nœuds de la tâche MPI doivent travailler sur les mêmes données de fichier. Le nœud principal crée un partage de fichiers auquel les nœuds enfants peuvent accéder s’ils s’exécutent sous le même compte d’utilisateur automatique.

L’extrait de code suivant définit l’étendue de l’utilisateur automatique sur l’étendue du pool pour une tâche dans Batch .NET. Le niveau d’élévation est omis, afin que la tâche s’exécute sous le compte standard d’utilisateur automatique au niveau du pool.

```csharp
task.UserIdentity = new UserIdentity(new AutoUserSpecification(scope: AutoUserScope.Pool));
```

## <a name="named-user-accounts"></a>Comptes d’utilisateur nommé

Vous pouvez définir des comptes d’utilisateur nommé lorsque vous créez un pool. Un compte d’utilisateur nommé possède un nom et un mot de passe que vous spécifiez. Vous pouvez spécifier le niveau d’élévation pour un compte d’utilisateur nommé. Pour les nœuds Linux, vous pouvez également fournir une clé privée SSH.

Il existe un compte d’utilisateur nommé sur tous les nœuds du pool et ce compte est disponible sur toutes les tâches en cours d’exécution sur ces nœuds. Vous pouvez définir n’importe quel nombre d’utilisateurs nommés pour un pool. Lorsque vous ajoutez une tâche ou une collection de tâches, vous pouvez indiquer que la tâche s’exécute sous l’un des comptes d’utilisateur nommé définis dans le pool.

Un compte d’utilisateur nommé est utile lorsque vous souhaitez exécuter toutes les tâches d’un travail sous le même compte d’utilisateur, mais que vous voulez les isoler des tâches qui s’exécutent simultanément dans d’autres travaux. Par exemple, vous pouvez créer un utilisateur nommé pour chaque travail, et exécuter les tâches de chaque travail sous ce compte d’utilisateur nommé. Chaque travail peut ensuite partager une clé secrète avec ses propres tâches, mais pas avec les tâches qui s’exécutent dans d’autres travaux.

Vous pouvez également utiliser un compte d’utilisateur nommé pour exécuter une tâche qui définit des autorisations sur des ressources externes, telles que des partages de fichiers. Avec un compte d’utilisateur nommé, vous pouvez contrôler l’identité de l’utilisateur et utiliser cette identité pour définir des autorisations.  

Les comptes d’utilisateur nommé activent le protocole SSH sans mot de passe entre les nœuds Linux. Vous pouvez utiliser un compte d’utilisateur nommé avec des nœuds Linux qui doivent exécuter des tâches de plusieurs instances. Chaque nœud du pool peut exécuter des tâches sous un compte d’utilisateur défini sur le pool entier. Pour plus d’informations sur les tâches multi-instances, consultez [Utiliser les tâches multi\-instances pour exécuter des applications MPI (Message Passing Interface) dans Batch](batch-mpi.md).

### <a name="create-named-user-accounts"></a>Créer des comptes d’utilisateur nommé

Pour créer des comptes d’utilisateur nommé dans Batch, ajoutez une collection de comptes d’utilisateur au pool. Les extraits de code suivants montrent comment créer des comptes d’utilisateur nommé dans .NET, Java et Python. Ces extraits de code montrent comment créer des comptes nommés admin et non-admin sur un pool. Les exemples montrent comment créer des pools à l’aide de la configuration du service cloud, mais en utilisant la même approche que lors de la création d’un pool Windows ou Linux à l’aide de la configuration de machine virtuelle.

#### <a name="batch-net-example-windows"></a>Exemple .NET Batch (Windows)

```csharp
CloudPool pool = null;
Console.WriteLine("Creating pool [{0}]...", poolId);

// Create a pool using the cloud service configuration.
pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: 3,
    virtualMachineSize: "standard_d1_v2",
    cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));

// Add named user accounts.
pool.UserAccounts = new List<UserAccount>
{
    new UserAccount("adminUser", "xyz123", ElevationLevel.Admin),
    new UserAccount("nonAdminUser", "123xyz", ElevationLevel.NonAdmin),
};

// Commit the pool.
await pool.CommitAsync();
```

#### <a name="batch-net-example-linux"></a>Exemple .NET Batch (Linux)

```csharp
CloudPool pool = null;

// Obtain a collection of all available node agent SKUs.
List<NodeAgentSku> nodeAgentSkus =
    batchClient.PoolOperations.ListNodeAgentSkus().ToList();

// Define a delegate specifying properties of the VM image to use.
Func<ImageReference, bool> isUbuntu1404 = imageRef =>
    imageRef.Publisher == "Canonical" &&
    imageRef.Offer == "UbuntuServer" &&
    imageRef.Sku.Contains("14.04");

// Obtain the first node agent SKU in the collection that matches
// Ubuntu Server 14.04. 
NodeAgentSku ubuntuAgentSku = nodeAgentSkus.First(sku =>
    sku.VerifiedImageReferences.Any(isUbuntu1404));

// Select an ImageReference from those available for node agent.
ImageReference imageReference =
    ubuntuAgentSku.VerifiedImageReferences.First(isUbuntu1404);

// Create the virtual machine configuration to use to create the pool.
VirtualMachineConfiguration virtualMachineConfiguration =
    new VirtualMachineConfiguration(imageReference, ubuntuAgentSku.Id);

Console.WriteLine("Creating pool [{0}]...", poolId);

// Create the unbound pool.
pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: 3,
    virtualMachineSize: "Standard_A1",
    virtualMachineConfiguration: virtualMachineConfiguration);
// Add named user accounts.
pool.UserAccounts = new List<UserAccount>
{
    new UserAccount(
        name: "adminUser",
        password: "xyz123",
        elevationLevel: ElevationLevel.Admin,
        linuxUserConfiguration: new LinuxUserConfiguration(
            uid: 12345,
            gid: 98765,
            sshPrivateKey: new Guid().ToString()
            )),
    new UserAccount(
        name: "nonAdminUser",
        password: "123xyz",
        elevationLevel: ElevationLevel.NonAdmin,
        linuxUserConfiguration: new LinuxUserConfiguration(
            uid: 45678,
            gid: 98765,
            sshPrivateKey: new Guid().ToString()
            )),
};

// Commit the pool.
await pool.CommitAsync();
```

#### <a name="batch-java-example"></a>Exemple Java Batch

```java
List<UserAccount> userList = new ArrayList<>();
userList.add(new UserAccount().withName(adminUserAccountName).withPassword(adminPassword).withElevationLevel(ElevationLevel.ADMIN));
userList.add(new UserAccount().withName(nonAdminUserAccountName).withPassword(nonAdminPassword).withElevationLevel(ElevationLevel.NONADMIN));
PoolAddParameter addParameter = new PoolAddParameter()
        .withId(poolId)
        .withTargetDedicatedNodes(POOL_VM_COUNT)
        .withVmSize(POOL_VM_SIZE)
        .withCloudServiceConfiguration(configuration)
        .withUserAccounts(userList);
batchClient.poolOperations().createPool(addParameter);
```

#### <a name="batch-python-example"></a>Exemple Python Batch

```python
users = [
    batchmodels.UserAccount(
        name='pool-admin',
        password='******',
        elevation_level=batchmodels.ElevationLevel.admin)
    batchmodels.UserAccount(
        name='pool-nonadmin',
        password='******',
        elevation_level=batchmodels.ElevationLevel.non_admin)
]
pool = batchmodels.PoolAddParameter(
    id=pool_id,
    user_accounts=users,
    virtual_machine_configuration=batchmodels.VirtualMachineConfiguration(
        image_reference=image_ref_to_use,
        node_agent_sku_id=sku_to_use),
    vm_size=vm_size,
    target_dedicated=vm_count)
batch_client.pool.add(pool)
```

### <a name="run-a-task-under-a-named-user-account-with-elevated-access"></a>Exécution d’une tâche sous un compte d’utilisateur nommé disposant d’un accès avec élévation de privilèges

Pour exécuter une tâche en tant qu’un utilisateur avec élévation de privilèges, définissez la propriété **UserIdentity** de la tâche sur un compte d’utilisateur nommé dont la propriété **ElevationLevel** a été définie sur `Admin`.

Cet extrait de code spécifie que la tâche doit s’exécuter sous un compte d’utilisateur nommé. Ce compte d’utilisateur nommé a été défini sur le pool au moment de la création du pool. Dans ce cas, le compte d’utilisateur nommé a été créé avec des autorisations d’administrateur :

```csharp
CloudTask task = new CloudTask("1", "cmd.exe /c echo 1");
task.UserIdentity = new UserIdentity(AdminUserAccountName);
```

## <a name="update-your-code-to-the-latest-batch-client-library"></a>Mettre à jour votre code vers la dernière bibliothèque du client Batch

La version du service Batch 2017-01-01.4.0 introduit une modification qui remplace la propriété **runElevated** disponible dans les versions précédentes par la propriété **userIdentity**. Les tableaux suivants fournissent un mappage simple que vous pouvez utiliser pour mettre à jour votre code à partir de versions antérieures des bibliothèques clientes.

### <a name="batch-net"></a>.NET Batch

| Si votre code utilise...                  | Mettez-le à jour vers...                                                                                                 |
|---------------------------------------|------------------------------------------------------------------------------------------------------------------|
| `CloudTask.RunElevated = true;`       | `CloudTask.UserIdentity = new UserIdentity(new AutoUserSpecification(elevationLevel: ElevationLevel.Admin));`    |
| `CloudTask.RunElevated = false;`      | `CloudTask.UserIdentity = new UserIdentity(new AutoUserSpecification(elevationLevel: ElevationLevel.NonAdmin));` |
| `CloudTask.RunElevated` non spécifié | Aucune mise à jour requise                                                                                               |

### <a name="batch-java"></a>Java Batch

| Si votre code utilise...                      | Mettez-le à jour vers...                                                                                                                       |
|-------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| `CloudTask.withRunElevated(true);`        | `CloudTask.withUserIdentity(new UserIdentity().withAutoUser(new AutoUserSpecification().withElevationLevel(ElevationLevel.ADMIN));`    |
| `CloudTask.withRunElevated(false);`       | `CloudTask.withUserIdentity(new UserIdentity().withAutoUser(new AutoUserSpecification().withElevationLevel(ElevationLevel.NONADMIN));` |
| `CloudTask.withRunElevated` non spécifié | Aucune mise à jour requise                                                                                                                     |

### <a name="batch-python"></a>Python Batch

| Si votre code utilise...                      | Mettez-le à jour vers...                                                                                                                       |
|-------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| `run_elevated=True`                       | `user_identity=user`, où <br />`user = batchmodels.UserIdentity(`<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`auto_user=batchmodels.AutoUserSpecification(`<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`elevation_level=batchmodels.ElevationLevel.admin))`                |
| `run_elevated=False`                      | `user_identity=user`, où <br />`user = batchmodels.UserIdentity(`<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`auto_user=batchmodels.AutoUserSpecification(`<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`elevation_level=batchmodels.ElevationLevel.non_admin))`             |
| `run_elevated` non spécifié | Aucune mise à jour requise                                                                                                                                  |

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur le [workflow et les ressources principales du service Batch](batch-service-workflow-features.md) telles que les pools, les nœuds, les travaux et les tâches.
- En savoir plus sur les [fichiers et répertoires](files-and-directories.md) dans Azure Batch.
