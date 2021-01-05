---
title: Chiffrement côté client avec Python
titleSuffix: Azure Storage
description: La bibliothèque cliente de stockage Azure pour Python prend en charge le chiffrement côté client pour assurer une sécurité maximale à vos applications Azure Storage.
services: storage
author: tamram
ms.service: storage
ms.devlang: python
ms.topic: how-to
ms.date: 12/04/2019
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: 511166e156591562b2120b58cc420f3fccd1d8c4
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96008926"
---
# <a name="client-side-encryption-with-python"></a>Chiffrement côté client avec Python

[!INCLUDE [storage-selector-client-side-encryption-include](../../../includes/storage-selector-client-side-encryption-include.md)]

## <a name="overview"></a>Vue d’ensemble
La [bibliothèque cliente de stockage Azure pour Python](https://pypi.python.org/pypi/azure-storage) prend en charge le chiffrement des données au sein des applications clientes, avant le chargement vers Azure Storage, et le déchiffrement des données pendant leur téléchargement vers le client.

> [!NOTE]
> La bibliothèque Azure Storage est disponible en version préliminaire.
> 
> 

## <a name="encryption-and-decryption-via-the-envelope-technique"></a>Chiffrement et déchiffrement via la technique d’enveloppe
Les processus de chiffrement et de déchiffrement s’effectuent selon la technique d’enveloppe.

### <a name="encryption-via-the-envelope-technique"></a>Chiffrement via la technique d’enveloppe
Le chiffrement via la technique d’enveloppe fonctionne de la façon suivante :

1. La bibliothèque cliente du stockage Azure génère une clé de chiffrement de contenu (CEK) qui est une clé symétrique à usage unique.
2. Les données utilisateur sont chiffrées à l'aide de cette clé de chiffrement de contenu.
3. La clé de chiffrement de contenu est ensuite encapsulée (chiffrée) à l’aide de la clé de chiffrement de clés (KEK). La clé de chiffrement de clés (KEK) est identifiée par un identificateur de clé et peut être une paire de clés asymétriques ou une clé symétrique gérée localement.
   La bibliothèque cliente du stockage n’a jamais accès à la clé de chiffrement de clés. Elle appelle l’algorithme d’encapsulage de clés fourni par la KEK. Si besoin est, les utilisateurs peuvent choisir d'utiliser des fournisseurs personnalisés pour l’encapsulage/le désencapsulage de clés.
4. Les données chiffrées sont ensuite téléchargées sur le service Azure Storage. La clé encapsulée avec des métadonnées de chiffrement supplémentaires est stockée en tant que métadonnées (sur un objet blob) ou interpolée avec les données chiffrées (messages de la file d’attente et entités de la table).

### <a name="decryption-via-the-envelope-technique"></a>Déchiffrement via la technique d’enveloppe
Le déchiffrement via la technique d’enveloppe fonctionne de la façon suivante :

1. La bibliothèque cliente suppose que l’utilisateur gère la clé de chiffrement de clés (KEK) localement. L’utilisateur n’est pas obligé de savoir quelle clé a été spécifiquement utilisée pour le chiffrement. Il est en effet possible d’utiliser et de configurer un programme de résolution de clés qui résout les différents identificateurs de clés.
2. La bibliothèque cliente télécharge les données chiffrées ainsi que  tout le  matériel de chiffrement stocké sur le service.
3. La clé de chiffrement de contenu (CEK) encapsulée est ensuite désencapsulée (déchiffrée) à l’aide de la clé de chiffrement de clés (KEK). Là encore, la bibliothèque cliente n’a pas accès à la clé de chiffrement de clés. Elle appelle simplement l’algorithme de désencapsulage du fournisseur personnalisé.
4. La clé de chiffrement de contenu (CEK) est alors utilisée pour déchiffrer les données utilisateur chiffrées.

## <a name="encryption-mechanism"></a>Mécanisme de chiffrement
La bibliothèque cliente du stockage utilise [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) pour chiffrer les données utilisateur. Plus précisément, le mode [CBC (Cipher Block Chaining)](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Cipher-block_chaining_.28CBC.29) avec AES. Chaque service fonctionnant un peu différemment, nous allons les étudier un par un ici.

### <a name="blobs"></a>Objets blob
La bibliothèque cliente prend actuellement en charge le chiffrement des objets blob entiers uniquement. Plus précisément, le chiffrement est pris en charge lorsque les utilisateurs utilisent les méthodes **create** _. Les téléchargements complets et les téléchargements de plages sont pris en charge, et la parallélisation du chargement et du téléchargement est disponible.

Au cours du chiffrement, la bibliothèque cliente génère un vecteur d’initialisation aléatoire (IV) de 16 octets avec une clé de chiffrement de contenu (CEK) aléatoire de 32 octets, puis effectue le chiffrement d’enveloppe des données d’objets blob à l’aide de ces informations. La clé de chiffrement de contenu encapsulée ainsi que des métadonnées de chiffrement supplémentaires sont ensuite stockées en tant que métadonnées d’objet blob en même temps que l’objet blob chiffré sur le service.

> [!WARNING]
> Si vous modifiez ou téléchargez vos propres métadonnées pour l’objet blob, vous devez vous assurer que ces métadonnées sont conservées. Si vous téléchargez les nouvelles métadonnées sans ces métadonnées, la clé de chiffrement de contenu encapsulée, le vecteur d’initialisation et d’autres métadonnées seront perdus et le contenu de l’objet blob ne sera plus jamais récupérable.
> 
> 

Le téléchargement d’un objet blob chiffré implique de récupérer le contenu de l’objet blob entier à l’aide des méthodes pratiques _*get**_ . La clé de chiffrement de contenu encapsulée est désencapsulée et utilisée en combinaison avec le vecteur d’initialisation (stocké en tant que métadonnées d’objet blob dans cet exemple) pour renvoyer les données déchiffrées aux utilisateurs.

Le téléchargement d’une plage arbitraire (méthodes _*get**_ avec paramètres de plage transmis) dans l’objet blob chiffré implique d’ajuster la plage fournie par les utilisateurs pour obtenir une petite quantité de données supplémentaires pouvant être utilisées pour déchiffrer la plage demandée.

Les objets blob de blocs et les objets blob de pages peuvent être chiffrés/déchiffrés à l’aide de ce schéma. Il n’existe actuellement aucune prise en charge pour le chiffrement des objets blob d’ajout.

### <a name="queues"></a>Files d’attente
Dans la mesure où les messages de la file d’attente peuvent avoir n’importe quel format, la bibliothèque cliente définit un format personnalisé qui inclut le vecteur d’initialisation (IV) et la clé de chiffrement de contenu (CEK) chiffrée dans le texte du message.

Au cours du chiffrement, la bibliothèque cliente génère un vecteur d’initialisation aléatoire de 16 octets avec une clé de chiffrement de contenu aléatoire de 32 octets, puis effectue le chiffrement d’enveloppe du texte du message de la file d’attente à l’aide de ces informations. La clé de chiffrement de contenu encapsulée et les métadonnées de chiffrement supplémentaires sont ensuite ajoutées au message chiffré de la file d’attente. Ce message modifié (illustré ci-dessous) est stocké sur le service.

```
<MessageText>{"EncryptedMessageContents":"6kOu8Rq1C3+M1QO4alKLmWthWXSmHV3mEfxBAgP9QGTU++MKn2uPq3t2UjF1DO6w","EncryptionData":{…}}</MessageText>
```

Au cours du déchiffrement, la clé encapsulée est extraite du message de la file d’attente et désencapsulée. Le vecteur d’initialisation est également extrait du message de la file d’attente et utilisé en combinaison avec la clé désencapsulée pour déchiffrer les données du message de la file d’attente. Notez que les métadonnées de chiffrement sont peu volumineuses (moins de 500 octets). Donc, même si elles entrent en compte dans la limite de 64 Ko du message de la file d’attente, leur impact reste facile à gérer.

### <a name="tables"></a>Tables
La bibliothèque cliente prend en charge le chiffrement des propriétés de l’entité pour les opérations d’insertion et de remplacement.

> [!NOTE]
> La fusion n’est pas prise en charge pour le moment. Si un sous-ensemble de propriétés a été chiffré précédemment à l’aide d’une clé différente, la fusion des nouvelles propriétés et la mise à jour des métadonnées entraîne une perte de données. L’opération de fusion nécessite d’effectuer des appels de service supplémentaires pour lire l’entité pré-existante à partir du service ou d’utiliser une nouvelle clé par propriété. Ces deux solutions ne conviennent pas pour des raisons de performances.
> 
> 

Le chiffrement des données d’une table fonctionne de la manière suivante :

1. Les utilisateurs spécifient les propriétés à chiffrer.
2. La bibliothèque cliente génère un vecteur d’initialisation (IV) aléatoire de 16 octets et une clé de chiffrement de contenu (CEK) aléatoire de 32 octets pour chaque entité, puis effectue le chiffrement d’enveloppe sur les propriétés individuelles à chiffrer en dérivant un nouveau vecteur d’initialisation par propriété. La propriété chiffrée est stockée en tant que données binaires.
3. La clé de chiffrement de contenu encapsulée et certaines métadonnées de chiffrement supplémentaires sont ensuite stockées sous la forme de deux propriétés réservées supplémentaires. La première propriété réservée (\_ClientEncryptionMetadata1) est une propriété de type chaîne qui conserve les informations sur le vecteur d’initialisation, la version et la clé encapsulée. La seconde propriété réservée (\_ClientEncryptionMetadata2) est une propriété de type binaire qui conserve les informations sur les propriétés chiffrées. Les informations contenues dans cette seconde propriété (\_ClientEncryptionMetadata2) sont elles-mêmes chiffrées.
4. En raison de ces propriétés réservées supplémentaires requises pour le chiffrement, les utilisateurs ne peuvent désormais avoir que 250 propriétés personnalisées au lieu de 252. La taille totale de l’entité doit être inférieure à 1 Mo.

   Notez que seules les propriétés de type chaîne peuvent être chiffrées. Si d’autres types de propriétés doivent être chiffrés, ils doivent être convertis en chaînes. Les chaînes chiffrées sont stockées sur le service en tant que propriétés binaires, et elles sont de nouveau converties en chaînes (chaînes brutes, et non des EntityProperties de type EdmType.STRING) après le déchiffrement.

   Pour les tables, outre la stratégie de chiffrement, les utilisateurs doivent spécifier les propriétés à chiffrer. Cela est possible en stockant ces propriétés dans des objets TableEntity dont le type est défini sur EdmType.STRING et en définissant le chiffrement sur true, ou en définissant la fonction encryption_resolver_function sur l’objet tableservice. Un programme de résolution de chiffrement est une fonction qui prend une clé de partition, une clé de ligne et un nom de propriété, puis renvoie une valeur booléenne indiquant si cette propriété doit être chiffrée. Au cours du chiffrement, la bibliothèque cliente utilise ces informations pour décider si une propriété doit être chiffrée lors de l’écriture en ligne. Le délégué fournit également la possibilité de définir la manière dont les propriétés sont chiffrées l’aide d’un programme logique. (Par exemple, si X, alors chiffrer la propriété A ; sinon chiffrer les propriétés A et B.) Notez qu’il n’est pas nécessaire de fournir ces informations lors de la lecture ou de l’interrogation des entités.

### <a name="batch-operations"></a>Opérations de traitement par lots
Une stratégie de chiffrement s’applique à toutes les lignes dans le lot. La bibliothèque cliente génère en interne un nouveau vecteur d’initialisation (IV) aléatoire et une clé de chiffrement de contenu (CEK) aléatoire par ligne dans le lot. Les utilisateurs peuvent également choisir de chiffrer différentes propriétés pour chaque opération dans le lot en définissant ce comportement dans le programme de résolution de chiffrement.
Si un lot est créé en tant que gestionnaire de contexte par le biais de la méthode tableservice batch(), la stratégie de chiffrement de tableservice est automatiquement appliquée au lot. Si un lot est créé explicitement par un appel du constructeur, la stratégie de chiffrement doit être transmise comme paramètre et ne doit pas être modifiée pour la durée de vie du lot.
Notez que les entités sont chiffrées, car elles sont insérées dans le lot à l’aide de la stratégie de chiffrement du lot (les entités ne sont pas chiffrées au moment de la validation du traitement par lots à l’aide de la stratégie de chiffrement tableservice).

### <a name="queries"></a>Requêtes
> [!NOTE]
> Comme les entités sont chiffrées, vous ne pouvez pas exécuter des requêtes qui filtrent sur une propriété chiffrée.  Si vous essayez, les résultats seront incorrects, car le service essaiera de comparer les données chiffrées aux données non chiffrées.
> 
> 
> Pour effectuer des opérations de requête, vous devez spécifier un programme de résolution de clé capable de résoudre toutes les clés dans le jeu de résultats. Si une entité contenue dans le résultat de la requête ne peut pas être résolue par rapport à un fournisseur, la bibliothèque cliente génère une erreur. Pour toute requête effectuant des projections côté serveur, la bibliothèque cliente ajoute par défaut les propriétés de métadonnées de chiffrement spéciales (\_ClientEncryptionMetadata1 et \_ClientEncryptionMetadata2) aux colonnes sélectionnées.
> 
> [!IMPORTANT]
> Tenez compte des points importants suivants quand vous utilisez le chiffrement côté client :
> 
> _ Pendant la lecture d’un objet blob chiffré ou l’écriture dans un objet blob chiffré, utilisez les commandes de chargement d’objets entiers et de téléchargement d’objets blob entiers/par plage. N’écrivez pas dans un objet blob chiffré à l’aide d’opérations de protocole telles que Put Block, Put Block List, Write Pages ou Clear Pages au risque d’endommager l’objet blob chiffré et de le rendre illisible.
> * Pour les tables, une contrainte similaire existe. Veillez à ne pas mettre à jour les propriétés chiffrées sans aussi mettre à jour les métadonnées de chiffrement.
> * Si vous définissez des métadonnées sur l’objet blob chiffré, vous risquez de remplacer les métadonnées relatives au chiffrement et nécessaires au déchiffrement, car la définition des métadonnées n’est pas additive. Cela est également vrai pour les instantanés : évitez de spécifier des métadonnées lors de la création d’un instantané d’objet blob chiffré. Si des métadonnées doivent être définies, veillez à appeler d’abord la méthode **get_blob_metadata** pour obtenir les métadonnées de chiffrement actuelles et éviter des écritures simultanées pendant la définition des métadonnées.
> * Activez l’indicateur **require_encryption** dans l’objet de service pour les utilisateurs qui doivent recourir uniquement à des données chiffrées. Pour plus d’informations, consultez la section ci-dessous.

La bibliothèque de stockage cliente attend l’utilisation de la clé de chiffrement de clés (KEK) et du programme de résolution de clé pour implémenter l’interface suivante. [d’d’Azure Key Vault](https://azure.microsoft.com/services/key-vault/) pour la gestion de la clé de chiffrement de clés (KEK) Python est en attente et sera intégrée ultérieurement à cette bibliothèque.

## <a name="client-api--interface"></a>API/Interface cliente
Après avoir créé un objet de service de stockage (c'est-à-dire blockblobservice), l’utilisateur peut affecter des valeurs aux champs qui constituent une stratégie de chiffrement : key_encryption_key, key_resolver_function et require_encryption. Les utilisateurs peuvent fournir uniquement une KEK, uniquement un programme de résolution ou les deux. key_encryption_key est le type de clé de base identifié à l’aide d’un identificateur de clé. Il fournit le programme logique pour l’encapsulage/le désencapsulage. key_resolver_function est utilisé pour résoudre une clé pendant le processus de déchiffrement. Elle retourne une KEK valide avec un identificateur de clé. Les utilisateurs ont ainsi la possibilité de choisir entre plusieurs clés gérées dans plusieurs emplacements.

La KEK doit implémenter les méthodes suivantes pour chiffrer des données :

* wrap_key(cek) : encapsule la clé de chiffrement de contenu (CEK) spécifiée (octets) à l’aide d’un algorithme choisi par l’utilisateur. Retourne la clé encapsulée.
* get_key_wrap_algorithm() : retourne l’algorithme utilisé pour encapsuler les clés.
* get_kid() : retourne l’ID de clé de type chaîne pour cette clé KEK.
  La KEK doit implémenter les méthodes suivantes pour déchiffrer des données :
* unwrap_key(cek, algorithm) : retourne la forme désencapsulée de la clé CEK spécifiée à l’aide de l’algorithme spécifié par la chaîne.
* get_kid() : retourne un ID de clé de type chaîne pour cette clé KEK.

Le programme de résolution de clé doit implémenter au moins une méthode qui, avec un ID de clé, retourne la KEK correspondante qui implémente l’interface ci-dessus. Seule cette méthode doit être affectée à la propriété key_resolver_function sur l’objet de service.

* Pour le chiffrement, la clé est toujours utilisée et l’absence de clé entraîne une erreur.
* Pour le déchiffrement :

  * S’il est spécifié, le programme de résolution de clé est appelé pour obtenir la clé. Si le programme de résolution est spécifié, mais ne comporte pas de mappage pour l’identificateur de clé, une erreur est générée.
  * Si le programme de résolution n’est pas spécifié, mais qu’une clé est spécifiée, celle-ci est utilisée si son identificateur correspond à l’identificateur de clé nécessaire. Si l’identificateur ne correspond pas, une erreur est générée.

    Les exemples de chiffrement dans azure.storage.samples présentent un scénario de bout en bout plus détaillé pour les objets blob, les files d’attente et les tables.
      Des exemples d’implémentation de la KEK et du résolveur de clé sont fournis dans les exemples de fichiers en tant que KeyWrapper et KeyResolver respectivement.

### <a name="requireencryption-mode"></a>Mode RequireEncryption
Les utilisateurs peuvent éventuellement activer un mode de fonctionnement dans lequel tous les chargements et téléchargements doivent être chiffrés. Dans ce mode, les tentatives de chargement de données sans une stratégie de chiffrement ou de téléchargement de données non chiffrées sur le service échouent sur le client. L’indicateur **require_encryption** sur l’objet de service contrôle ce comportement.

### <a name="blob-service-encryption"></a>Chiffrement du service BLOB
Définissez les champs de stratégie de chiffrement sur l’objet blockblobservice. Tout le reste est géré par la bibliothèque cliente en interne.

```python
# Create the KEK used for encryption.
# KeyWrapper is the provided sample implementation, but the user may use their own object as long as it implements the interface above.
kek = KeyWrapper('local:key1')  # Key identifier

# Create the key resolver used for decryption.
# KeyResolver is the provided sample implementation, but the user may use whatever implementation they choose so long as the function set on the service object behaves appropriately.
key_resolver = KeyResolver()
key_resolver.put_key(kek)

# Set the KEK and key resolver on the service object.
my_block_blob_service.key_encryption_key = kek
my_block_blob_service.key_resolver_funcion = key_resolver.resolve_key

# Upload the encrypted contents to the blob.
my_block_blob_service.create_blob_from_stream(
    container_name, blob_name, stream)

# Download and decrypt the encrypted contents from the blob.
blob = my_block_blob_service.get_blob_to_bytes(container_name, blob_name)
```

### <a name="queue-service-encryption"></a>Chiffrement du service de File d’attente
Définissez les champs de stratégie de chiffrement sur l’objet queueservice. Tout le reste est géré par la bibliothèque cliente en interne.

```python
# Create the KEK used for encryption.
# KeyWrapper is the provided sample implementation, but the user may use their own object as long as it implements the interface above.
kek = KeyWrapper('local:key1')  # Key identifier

# Create the key resolver used for decryption.
# KeyResolver is the provided sample implementation, but the user may use whatever implementation they choose so long as the function set on the service object behaves appropriately.
key_resolver = KeyResolver()
key_resolver.put_key(kek)

# Set the KEK and key resolver on the service object.
my_queue_service.key_encryption_key = kek
my_queue_service.key_resolver_funcion = key_resolver.resolve_key

# Add message
my_queue_service.put_message(queue_name, content)

# Retrieve message
retrieved_message_list = my_queue_service.get_messages(queue_name)
```

### <a name="table-service-encryption"></a>Chiffrement du service de Table
En plus de créer une stratégie de chiffrement et de la définir dans les options de requête, vous devez spécifier une **encryption_resolver_function** dans le **tableservice** ou définir l’attribut de chiffrement sur EntityProperty.

### <a name="using-the-resolver"></a>Utilisation du programme de résolution

```python
# Create the KEK used for encryption.
# KeyWrapper is the provided sample implementation, but the user may use their own object as long as it implements the interface above.
kek = KeyWrapper('local:key1')  # Key identifier

# Create the key resolver used for decryption.
# KeyResolver is the provided sample implementation, but the user may use whatever implementation they choose so long as the function set on the service object behaves appropriately.
key_resolver = KeyResolver()
key_resolver.put_key(kek)

# Define the encryption resolver_function.


def my_encryption_resolver(pk, rk, property_name):
    if property_name == 'foo':
        return True
    return False


# Set the KEK and key resolver on the service object.
my_table_service.key_encryption_key = kek
my_table_service.key_resolver_funcion = key_resolver.resolve_key
my_table_service.encryption_resolver_function = my_encryption_resolver

# Insert Entity
my_table_service.insert_entity(table_name, entity)

# Retrieve Entity
# Note: No need to specify an encryption resolver for retrieve, but it is harmless to leave the property set.
my_table_service.get_entity(
    table_name, entity['PartitionKey'], entity['RowKey'])
```

### <a name="using-attributes"></a>Utilisation des attributs
Comme mentionné ci-dessus, une propriété peut être marquée pour le chiffrement en la stockant dans un objet EntityProperty et en définissant le champ de chiffrement.

```python
encrypted_property_1 = EntityProperty(EdmType.STRING, value, encrypt=True)
```

## <a name="encryption-and-performance"></a>Chiffrement et performances
Notez que le chiffrement de vos données de stockage affecte les performances. La clé de contenu et le vecteur d’initialisation doivent être générés, le contenu proprement dit doit être chiffré et des métadonnées supplémentaires doivent être mises en forme et chargées. Cette surcharge varie selon la quantité de données chiffrées. Nous recommandons de tester systématiquement les performances des applications au cours du développement.

## <a name="next-steps"></a>Étapes suivantes
* Télécharger la [bibliothèque cliente Azure Storage pour le package Java PyPi](https://pypi.python.org/pypi/azure-storage)
* Télécharger la [bibliothèque cliente Azure Storage pour le code source Python à partir de GitHub](https://github.com/Azure/azure-storage-python)
