# dev-rapid

Facilitate integrations between developer systems and data collectors.

## What is it?

Dev-rapid is a Kafka topic that contains events from systems that NAV developers use.

## Which data is available?

| Description | Produced by | Protobuf format |
|-------------|-------------|-----------------|
| Application deployment events | [Naiserator](https://github.com/nais/naiserator) and [Fasit](https://github.com/navikt/fasit) | [deployment.Event](https://github.com/navikt/protos/blob/master/deployment/event.proto) |

## Message format

All data on dev-rapid is published in the Protobuf format.  Protobuf is a
strongly typed message format that is extremely efficient both in terms of
speed and data size.  Libraries for reading them are available for most
programming languages.

### Compile Protobuf code

To use Protobuf messages in your code, you have to convert the message format
(in the form of a `.proto` file) to a class type.  This is done by the `protoc`
compiler. Compilers exist for virtually all programming languages.

The process is documented in the
[Protobuf API reference](https://developers.google.com/protocol-buffers/docs/reference/overview).
In short, this is usually done by a command such as:

```
wget https://path/to/protobuf/file.proto
protoc --proto_path=. --java_out=src file.proto
```

## Get access

1) By pull request, add your application to the [topic spec](topic.yaml) with the correct permissions (`read`, `write`, or `readwrite`).
2) [Configure your application](https://doc.nais.io/addons/kafka/#accessing-topics-from-an-application) according to the NAIS documentation.
3) Consume or produce using the topic `aura.dev-rapid`. For development purposes, use the pool `nav-dev`. This pool is only available in development clusters. For production use, use the pool `nav-infrastructure`. This pool is available from both development and production clusters.

## Consuming data

Compile Protobuf code for your application, see the section above.

Data consumers MUST NOT assume that all fields will be populated with data,
and MUST always do validation of fields before processing.

When consuming messages, try to deserialize the message into the message
format(s) you care about.  If the deserialization fails, the message is in a
different format, and you should silently drop the message and acknowledge your
consumer offset.

## Producing data

Define your message type in the Protobuf format.
See [Protobuf Tutorials](https://developers.google.com/protocol-buffers/docs/tutorials)
and [Protobuf Language Guide](https://developers.google.com/protocol-buffers/docs/proto)
for details.

Commit the `.proto` file to a repository, and add a reference in the table
above in the section _Which data is available_.

Compile Protobuf code for your application, see the section above.

Serialize your data in the Protobuf binary format, and produce it on the Kafka topic.

### Notes on compatibility

Protobuf messages are forwards and backwards compatible, and will continue to
work with older and newer clients provided that you follow these golden rules:

1) Message name, field types, field names, or enumerations MUST NOT change.
2) Messages or fields MUST NOT be removed.

Breaking these rules will break all consumers of your message. Don't do it.

In practice, data formats change over time. You can always add new fields with new data, and consumers will continue to work.
If a field is no longer in use, and you want to remove it, you can do it by changing the field type to `reserved`,
see the [note in the Protobuf Language Guide](https://developers.google.com/protocol-buffers/docs/proto3#reserved).
