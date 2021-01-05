---
title: Azure Event Hubs – Échanger des événements à l’aide de différents protocoles
description: Cet article explique comment les contrôles serveur consommateurs et les producteurs qui utilisent des protocoles différents (AMQP, Apache Kafka et HTTPS) peuvent échanger des événements avec Azure Event Hubs.
ms.topic: article
ms.date: 06/23/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: cbc6999e3ede73b948ce034769966922b4b0f282
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89010317"
---
# <a name="exchange-events-between-consumers-and-producers-that-use-different-protocols-amqp-kafka-and-https"></a>Échanger des événements entre des consommateurs et des producteurs qui utilisent des protocoles différents : AMQP, Kafka et HTTPS
Azure Event Hubs prend en charge trois protocoles pour les producteurs et les consommateurs : AMQP, Kafka et HTTPS. Chacun de ces protocoles dispose de sa propre méthode pour représenter les messages, donc bien la question suivante se pose naturellement : si une application envoie des événements à Event Hub dans un protocole et les consomme selon un autre protocole, à quoi ressemblent les différentes parties et valeurs de l’événement lorsqu’ils arrivent chez le contrôle serveur consommateur ? Cet article décrit les meilleures pratiques pour le producteur et le contrôle serveur consommateur pour vous assurer que les valeurs au sein d’un événement sont correctement interprétées par l’application qui les reçoit.

Plus précisément, les conseils de cet article décrivent ces clients, avec les versions répertoriées utilisées pour développer les extraits de code :

* Client Java Kafka (version 1.1.1 à partir de https://www.mvnrepository.com/artifact/org.apache.kafka/kafka-clients)
* Client Microsoft Azure Event Hubs pour Java (version 1.1.0 à partir de https://github.com/Azure/azure-event-hubs-java)
* Client Microsoft Azure Event Hubs pour .NET (version 2.1.0 à partir de https://github.com/Azure/azure-event-hubs-dotnet)
* Microsoft Azure Service Bus (version 5.0.0 à partir de https://www.nuget.org/packages/WindowsAzure.ServiceBus)
* HTTPS (producteurs uniquement)

Les autres clients AMQP peuvent se comporter de façon légèrement différente. AMQP dispose d’un système de types bien défini, mais les spécificités de la sérialisation des types spécifiques à chaque langage vers et à partir de ce système de types dépend du client, tout comme la façon dont le client fournit l’accès aux différentes parties du message AMQP.

## <a name="event-body"></a>Corps de l’événement
Tous les clients Microsoft AMQP représentent le corps de l’événement sous la forme d’un jeu d’octets non interprété. Une application de production transmet une séquence d’octets au client et une application de consommation reçoit cette même séquence du client. L’interprétation de la séquence d’octets s’opère dans le code de l’application.

Lors de l’envoi d’un événement via HTTPS, le corps de l’événement est le contenu de l’opération POST, qui est également traité comme des octets non interprétés. Il est facile d’obtenir le même état dans un producteur ou un contrôle serveur consommateur Kafka avec les composants ByteArraySerializer et ByteArrayDeserializer fournis, comme indiqué dans le code suivant :

### <a name="kafka-byte-producer"></a>Kafka byte[] producer

```java
final Properties properties = new Properties();
// add other properties
properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, ByteArraySerializer.class.getName());

final KafkaProducer<Long, byte[]> producer = new KafkaProducer<Long, byte[]>(properties);

final byte[] eventBody = new byte[] { 0x01, 0x02, 0x03, 0x04 };
ProducerRecord<Long, byte[]> pr =
    new ProducerRecord<Long, byte[]>(myTopic, myPartitionId, myTimeStamp, eventBody);
```

### <a name="kafka-byte-consumer"></a>Kafka byte[] consumer
```java
final Properties properties = new Properties();
// add other properties
properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, ByteArrayDeserializer.class.getName());

final KafkaConsumer<Long, byte[]> consumer = new KafkaConsumer<Long, byte[]>(properties);

ConsumerRecord<Long, byte[]> cr = /* receive event */
// cr.value() is a byte[] with values { 0x01, 0x02, 0x03, 0x04 }
```

Ce code crée un pipeline d’octets transparent entre les deux parties de l’application et permet au développeur de l’application de manuellement sérialiser et désérialiser comme il le souhaite, y décider de procéder à la désérialisation lors de l’exécution, par exemple en fonction du type ou des informations de l’expéditeur dans les propriétés définies par l’utilisateur de l’événement.

Les applications qui ont un seul type de corps d’événement fixe peuvent utiliser d’autres sérialiseurs et désérialiseurs Kafka pour convertir en toute transparence les données. Imaginez par exemple une application qui utilise JSON. La construction et l’interprétation de la chaîne JSON se produit au niveau de l’application. Au niveau Event Hubs, le corps de l’événement est toujours une chaîne, une séquence d’octets représentant les caractères avec un encodage UTF-8. Dans ce cas, le producteur ou le contrôle serveur consommateur Kafka peut tirer parti des composants StringSerializer ou StringDeserializer fournis comme indiqué dans le code suivant :

### <a name="kafka-utf-8-string-producer"></a>Kafka UTF-8 string producer
```java
final Properties properties = new Properties();
// add other properties
properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

final KafkaProducer<Long, String> producer = new KafkaProducer<Long, String>(properties);

final String exampleJson = "{\"name\":\"John\", \"number\":9001}";
ProducerRecord<Long, String> pr =
    new ProducerRecord<Long, String>(myTopic, myPartitionId, myTimeStamp, exampleJson);
```

### <a name="kafka-utf-8-string-consumer"></a>Kafka UTF-8 string consumer
```java
final Properties properties = new Properties();
// add other properties
properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

final KafkaConsumer<Long, String> consumer = new KafkaConsumer<Long, String>(properties);

ConsumerRecord<Long, Bytes> cr = /* receive event */
final String receivedJson = cr.value();
```

Côté AMQP, Java et .NET offrent de façon intégrée la conversion des chaînes vers et depuis les séquences d’octets UTF-8. Les clients Microsoft AMQP représentent les événements comme classe EventData. Les exemples suivants montrent comment sérialiser une chaîne UTF-8 dans le corps d’un événement EventData dans producteur d’AMQP et comment désérialiser le corps d’un événement EventData dans une chaîne UTF-8 dans un contrôle serveur consommateur AMQP.

### <a name="java-amqp-utf-8-string-producer"></a>Java AMQP UTF-8 string producer
```java
final String exampleJson = "{\"name\":\"John\", \"number\":9001}";
final EventData ed = EventData.create(exampleJson.getBytes(StandardCharsets.UTF_8));
```

### <a name="java-amqp-utf-8-string-consumer"></a>Java AMQP UTF-8 string consumer
```java
EventData ed = /* receive event */
String receivedJson = new String(ed.getBytes(), StandardCharsets.UTF_8);
```

### <a name="c-net-utf-8-string-producer"></a>C# .NET UTF-8 string producer
```csharp
string exampleJson = "{\"name\":\"John\", \"number\":9001}";
EventData working = new EventData(Encoding.UTF8.GetBytes(exampleJson));
```

### <a name="c-net-utf-8-string-consumer"></a>C# .NET UTF-8 string consumer
```csharp
EventData ed = /* receive event */

// getting the event body bytes depends on which .NET client is used
byte[] bodyBytes = ed.Body.Array;  // Microsoft Azure Event Hubs Client for .NET
// byte[] bodyBytes = ed.GetBytes(); // Microsoft Azure Service Bus

string receivedJson = Encoding.UTF8.GetString(bodyBytes);
```

Comme Kafka est open source, le développeur de l’application peut inspecter l’implémentation de n’importe quel sérialiseur ou désérialiseur et implémenter le code qui produit ou consomme une séquence compatible d’octets côté AMQP.

## <a name="event-user-properties"></a>Propriétés de l’utilisateur d’événement

Les propriétés définies par l’utilisateur peuvent être définies et récupérées à partir des deux clients AMQP (dans les clients Microsoft AMQP, elles sont appelées propriétés) et Kafka (où elles sont appelées en-têtes). Les expéditeurs HTTPS peuvent définir des propriétés utilisateur sur un événement en les fournissant sous forme d’en-têtes HTTP dans l’opération POST. Toutefois, Kafka traite le corps de l’événement et les valeurs de l’en-tête d’événement en tant que séquences d’octets. Tandis que les clients AMQP, les valeurs de propriété comportent des types qui sont communiqués en encodant les valeurs de propriété selon le système de types AMQP.

Le HTTPS est un cas à part. Au moment de l’envoi, toutes les valeurs de propriété sont en texte UTF-8. Le service Event Hubs effectue une quantité limitée d’interprétation pour convertir les valeurs des propriétés concernées codées en entiers signés AMQP 32 bits et 64 bits, en nombres à virgule flottante 64 bits et en valeurs booléennes. Toute valeur de propriété qui ne correspond pas à un de ces types est traitée comme une chaîne.

En combinant ces approches pour définir le type de propriété, le contrôle serveur consommateur Kafka voit la séquence d’octets brute encodés en AMQP, y compris les informations de type AMQP. Tandis que le contrôle serveur consommateur AMQP voit la séquence d’octets non typée envoyée par le producteur Kafka, qui doit alors être analysée par l’application.

Pour les contrôles serveur consommateurs Kafka qui reçoivent les propriétés de producteurs AMQP ou HTTPS, utilisez la classe AmqpDeserializer, qui est modélisée sur les autres désérialiseurs de l’écosystème Kafka. Elle interprète les informations de type dans les séquences d’octets encodés en AMQP pour désérialiser les octets de données en un type Java.

Comme meilleure pratique, nous vous conseillons d’inclure une propriété dans les messages envoyés via AMQP ou HTTPS. Le contrôle serveur consommateur Kafka peut l’utiliser pour déterminer si les valeurs d’en-tête nécessitent une désérialisation AMQP. La valeur de la propriété n’est pas importante. Il a juste besoin d’un nom connu que le contrôle serveur consommateur Kafka peut retrouver dans la liste des en-têtes et ajuster son comportement en conséquence.

### <a name="amqp-to-kafka-part-1-create-and-send-an-event-in-c-net-with-properties"></a>AMQP à Kafka partie 1 : créer et envoyer un événement en C# (.NET) avec des propriétés
```csharp
// Create an event with properties "MyStringProperty" and "MyIntegerProperty"
EventData working = new EventData(Encoding.UTF8.GetBytes("an event body"));
working.Properties.Add("MyStringProperty", "hello");
working.Properties.Add("MyIntegerProperty", 1234);

// BEST PRACTICE: include a property which indicates that properties will need AMQP deserialization
working.Properties.Add("AMQPheaders", 0);
```

### <a name="amqp-to-kafka-part-2-use-amqpdeserializer-to-deserialize-those-properties-in-a-kafka-consumer"></a>AMQP à Kafka partie 2 : utiliser AmqpDeserializer pour désérialiser ces propriétés dans un contrôle serveur consommateur Kafka
```java
final AmqpDeserializer amqpDeser = new AmqpDeserializer();

ConsumerRecord<Long, Bytes> cr = /* receive event */
final Header[] headers = cr.headers().toArray();

final Header headerNamedMyStringProperty = /* find header with key "MyStringProperty" */
final Header headerNamedMyIntegerProperty = /* find header with key "MyIntegerProperty" */
final Header headerNamedAMQPheaders = /* find header with key "AMQPheaders", or null if not found */

// BEST PRACTICE: detect whether AMQP deserialization is needed
if (headerNamedAMQPheaders != null) {
    // The deserialize() method requires no prior knowledge of a property's type.
    // It returns Object and the application can check the type and perform a cast later.
    Object propertyOfUnknownType = amqpDeser.deserialize("topicname", headerNamedMyStringProperty.value());
    if (propertyOfUnknownType instanceof String) {
        final String propertyString = (String)propertyOfUnknownType;
        // do work here
    }
    propertyOfUnknownType = amqpDeser.deserialize("topicname", headerNamedMyIntegerProperty.value());
    if (propertyOfUnknownType instanceof Integer) {
        final Integer propertyInt = (Integer)propertyOfUnknownType;
        // do work here
    }
} else {
    /* event sent via Kafka, interpret header values the Kafka way */
}
```

Si l’application connaît le type attendu d’une propriété, il existe des méthodes de désérialisation qui ne nécessitent pas de cast par la suite, mais elles génèrent une erreur si la propriété ne correspond pas au type attendu.

### <a name="amqp-to-kafka-part-3-a-different-way-of-using-amqpdeserializer-in-a-kafka-consumer"></a>AMQP à Kafka partie 3 : une autre utilisation d’AmqpDeserializer dans un contrôle serveur consommateur Kafka
```java
// BEST PRACTICE: detect whether AMQP deserialization is needed
if (headerNamedAMQPheaders != null) {
    // Property "MyStringProperty" is expected to be of type string.
    try {
        final String propertyString = amqpDeser.deserializeString(headerNamedMyStringProperty.value());
        // do work here
    }
    catch (IllegalArgumentException e) {
        // property was not a string
    }

    // Property "MyIntegerProperty" is expected to be a signed integer type.
    // The method returns long because long can represent the value range of all AMQP signed integer types.
    try {
        final long propertyLong = amqpDeser.deserializeSignedInteger(headerNamedMyIntegerProperty.value());
        // do work here
    }
    catch (IllegalArgumentException e) {
        // property was not a signed integer
    }
} else {
    /* event sent via Kafka, interpret header values the Kafka way */
}
```

L’opération dans l’autre sens est plus compliquée, étant donné que les en-têtes définis par le producteur Kafka sont toujours visibles par le contrôle serveur consommateur AMQP comme octets bruts (tapez org.apache.qpid.proton.amqp.Binary pour le client Microsoft Azure Event Hubs pour Java ou `System.Byte[]` pour les clients Microsoft .NET AMQP). Le plus simple consiste à utiliser un des sérialiseurs fournis par Kafka pour générer les octets pour les valeurs d’en-tête côté producteur Kafka et ensuite à écrire un code compatible avec la désérialisation côté client AMQP.

Comme pour AMQP vers Kafka, la meilleure pratique que nous recommandons est d’inclure une propriété dans les messages envoyés via Kafka. Le contrôle serveur consommateur AMQP peut utiliser la propriété pour déterminer si les valeurs d’en-tête nécessitent une désérialisation. La valeur de la propriété n’est pas importante. Il a juste besoin d’un nom connu que le contrôle serveur consommateur AMQP peut retrouver dans la liste des en-têtes et ajuster son comportement en conséquence. Si le producteur Kafka ne peut pas être modifié, il est également possible que l’application de consommation vérifie si la valeur de la propriété est de type binaire ou de type octets et tente une désérialisation en fonction du type.

### <a name="kafka-to-amqp-part-1-create-and-send-an-event-from-kafka-with-properties"></a>Kafka à AMQP partie 1 : créer et envoyer un événement depuis Kafka avec des propriétés
```java
final String topicName = /* topic name */
final ProducerRecord<Long, String> pr = new ProducerRecord<Long, String>(topicName, /* other arguments */);
final Headers h = pr.headers();

// Set headers using Kafka serializers
IntegerSerializer intSer = new IntegerSerializer();
h.add("MyIntegerProperty", intSer.serialize(topicName, 1234));

LongSerializer longSer = new LongSerializer();
h.add("MyLongProperty", longSer.serialize(topicName, 5555555555L));

ShortSerializer shortSer = new ShortSerializer();
h.add("MyShortProperty", shortSer.serialize(topicName, (short)22222));

FloatSerializer floatSer = new FloatSerializer();
h.add("MyFloatProperty", floatSer.serialize(topicName, 1.125F));

DoubleSerializer doubleSer = new DoubleSerializer();
h.add("MyDoubleProperty", doubleSer.serialize(topicName, Double.MAX_VALUE));

StringSerializer stringSer = new StringSerializer();
h.add("MyStringProperty", stringSer.serialize(topicName, "hello world"));

// BEST PRACTICE: include a property which indicates that properties will need deserialization
h.add("RawHeaders", intSer.serialize(0));
```

### <a name="kafka-to-amqp-part-2-manually-deserialize-those-properties-in-c-net"></a>Kafka à AMQP partie 2 : désérialiser manuellement ces propriétés dans C# (.NET)
```csharp
EventData ed = /* receive event */

// BEST PRACTICE: detect whether manual deserialization is needed
if (ed.Properties.ContainsKey("RawHeaders"))
{
    // Kafka serializers send bytes in big-endian order, whereas .NET on x86/x64 is little-endian.
    // Therefore it is frequently necessary to reverse the bytes before further deserialization.

    byte[] rawbytes = ed.Properties["MyIntegerProperty"] as System.Byte[];
    if (BitConverter.IsLittleEndian)
    {
            Array.Reverse(rawbytes);
    }
    int myIntegerProperty = BitConverter.ToInt32(rawbytes, 0);

    rawbytes = ed.Properties["MyLongProperty"] as System.Byte[];
    if (BitConverter.IsLittleEndian)
    {
            Array.Reverse(rawbytes);
    }
    long myLongProperty = BitConverter.ToInt64(rawbytes, 0);

    rawbytes = ed.Properties["MyShortProperty"] as System.Byte[];
    if (BitConverter.IsLittleEndian)
    {
            Array.Reverse(rawbytes);
    }
    short myShortProperty = BitConverter.ToInt16(rawbytes, 0);

    rawbytes = ed.Properties["MyFloatProperty"] as System.Byte[];
    if (BitConverter.IsLittleEndian)
    {
            Array.Reverse(rawbytes);
    }
    float myFloatProperty = BitConverter.ToSingle(rawbytes, 0);

    rawbytes = ed.Properties["MyDoubleProperty"] as System.Byte[];
    if (BitConverter.IsLittleEndian)
    {
            Array.Reverse(rawbytes);
    }
    double myDoubleProperty = BitConverter.ToDouble(rawbytes, 0);

    rawbytes = ed.Properties["MyStringProperty"] as System.Byte[];
string myStringProperty = Encoding.UTF8.GetString(rawbytes);
}
```

### <a name="kafka-to-amqp-part-3-manually-deserialize-those-properties-in-java"></a>Kafka à AMQP partie 3 : désérialiser manuellement ces propriétés dans Java
```java
final EventData ed = /* receive event */

// BEST PRACTICE: detect whether manual deserialization is needed
if (ed.getProperties().containsKey("RawHeaders")) {
    byte[] rawbytes =
        ((org.apache.qpid.proton.amqp.Binary)ed.getProperties().get("MyIntegerProperty")).getArray();
    int myIntegerProperty = 0;
    for (byte b : rawbytes) {
        myIntegerProperty <<= 8;
        myIntegerProperty |= ((int)b & 0x00FF);
    }

    rawbytes = ((org.apache.qpid.proton.amqp.Binary)ed.getProperties().get("MyLongProperty")).getArray();
    long myLongProperty = 0;
    for (byte b : rawbytes) {
        myLongProperty <<= 8;
        myLongProperty |= ((long)b & 0x00FF);
    }

    rawbytes = ((org.apache.qpid.proton.amqp.Binary)ed.getProperties().get("MyShortProperty")).getArray();
    short myShortProperty = (short)rawbytes[0];
    myShortProperty <<= 8;
    myShortProperty |= ((short)rawbytes[1] & 0x00FF);

    rawbytes = ((org.apache.qpid.proton.amqp.Binary)ed.getProperties().get("MyFloatProperty")).getArray();
    int intbits = 0;
    for (byte b : rawbytes) {
        intbits <<= 8;
        intbits |= ((int)b & 0x00FF);
    }
    float myFloatProperty = Float.intBitsToFloat(intbits);

    rawbytes = ((org.apache.qpid.proton.amqp.Binary)ed.getProperties().get("MyDoubleProperty")).getArray();
    long longbits = 0;
    for (byte b : rawbytes) {
        longbits <<= 8;
        longbits |= ((long)b & 0x00FF);
    }
    double myDoubleProperty = Double.longBitsToDouble(longbits);

    rawbytes = ((org.apache.qpid.proton.amqp.Binary)ed.getProperties().get("MyStringProperty")).getArray();
String myStringProperty = new String(rawbytes, StandardCharsets.UTF_8);
}
```

## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez appris à diffuser en streaming dans Event Hubs sans changer vos clients de protocole ni exécuter vos propres clusters. Pour plus d’informations sur Event Hubs et sur Event Hubs pour Kafka, consultez les articles suivants :  

* [En savoir plus sur Event Hubs](./event-hubs-about.md)
* [En savoir plus sur Event Hubs pour Kafka](event-hubs-for-kafka-ecosystem-overview.md)
* [Explorer d’autres exemples sur le site GitHub Event Hubs pour Kafka](https://github.com/Azure/azure-event-hubs-for-kafka)
* Utilisez [MirrorMaker](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=27846330) pour [diffuser en streaming des événements à partir de Kafka en local vers Event Hubs dans le cloud](event-hubs-kafka-mirror-maker-tutorial.md).
* Découvrez-en plus sur le streaming vers Event Hubs avec des [applications Kafka natives](event-hubs-quickstart-kafka-enabled-event-hubs.md), [Apache Flink](event-hubs-kafka-flink-tutorial.md) ou [Akka Streams](event-hubs-kafka-akka-streams-tutorial.md).
