# Apache Kafka AMI

AMI that should be used to create virtual machines with Apache Kafka
installed.

## Synopsis

This script will create an AMI with Apache Kafka installed and with all of
the required initialization scripts.

The AMI resulting from this script should be the one used to instantiate a
Kafka server (standalone or cluster).

## Getting Started

There are a couple of things needed for the script to work.

### Prerequisites

Packer and AWS Command Line Interface tools need to be installed on your local
computer.
To build a base image you have to know the id of the latest Debian AMI files
for the region where you wish to build the AMI.

#### Packer

Packer installation instructions can be found
[here](https://www.packer.io/docs/installation.html).

#### AWS Command Line Interface

AWS Command Line Interface installation instructions can be found [here](http://docs.aws.amazon.com/cli/latest/userguide/installing.html)

#### Debian AMI's

A list of all the Debian AMI id's can be found at the debian official page:
[Debian oficial Amazon EC2 Images](https://wiki.debian.org/Cloud/AmazonEC2Image/)

### Usage

In order to create the AMI using this packer template you need to provide a
few options.

```
Usage:
  packer build \
    -var 'aws_access_key=AWS_ACCESS_KEY' \
    -var 'aws_secret_key=<AWS_SECRET_KEY>' \
    -var 'aws_region=<AWS_REGION>' \
    -var 'aws_base_ami=<BASE_IMAGE>' \
    -var 'kafka_version=<KAFKA_VERSION>' \
    [-var 'option=value'] \
    kafka.json
```

#### Script Options

- `aws_access_key` - *[required]* The AWS access key.
- `aws_ami_name` - The AMI name (default value: "kafka").
- `aws_ami_name_prefix` - Prefix for the AMI name (default value: "").
- `aws_base_ami` - *[required]* The AWS base AMI id to use. See [here](https://wiki.debian.org/Cloud/AmazonEC2Image/) for a list of available options.
- `aws_instance_type` - The instance type to use for the build (default value: "t2.micro").
- `aws_region` - *[required]* The regions were the build will be performed.
- `aws_secret_key` - *[required]* The AWS secret key.
- `java_build_number` - Java build number (default value: "15").
- `java_major_version` - Java major version (default value: "8").
- `java_update_version` - Java update version (default value: "112").
- `kafka_scala_version` - Kafka Scala version (default value: "2.11").
- `kafka_version` - *[required]* Kafka version.
- `system_locale` - Locale for the system (default value: "en_US").
- `zookeeper_version` - Zookeeper version (default value: "3.4.9").

### Instantiate a Cluster

In order to end up with a functional Kafka Cluster some configurations have to
be performed after instantiating the servers.

To help perform those configurations a small script is included on the AWS
image. The script is called **kafka_config**.

A Zookeeper instance or cluster (for production environments) is required to
instantiate a Kafka cluster.

If required, a Zookeeper server/node can be started within this image (see
below for more information) however, for production environments, it is
recommended to use a dedicated and separated Zookeeper instance or cluster.

#### Kafka Configuration Script

The script can and should be used to set some of the Kafka options as well as
setting the Kafka service to start at boot.

```
Usage: kafka_config [options]
```

##### Options

* `-a <ADDRESS>` - Sets the Kafka broker advertised address (default value is 'localhost').
* `-D` - Disables the Kafka service from start at boot time.
* `-E` - Enables the Kafka service to start at boot time.
* `-i <ID>` - Sets the Kafka broker ID (default value is '0').
* `-m <MEMORY>` - Sets Kafka maximum heap size. Values should be provided following the same Java heap nomenclature.
* `-S` - Starts the Kafka service after performing the required configurations (if any given).
* `-W <SECONDS>` - Waits the specified amount of seconds before starting the Kafka service (default value is '0').
* `-z <ENDPOINT>` - Sets a Zookeeper server endpoint to be used by the Kafka broker (defaut value is 'localhost:2181'). Several Zookeeper endpoints can be set by either using extra `-z` options or if separated with a comma on the same `-z` option.

#### Configuring a Kafka Broker

To prepare an instance to act as a Kafka broker the following steps need to be
performed.

Run the configuration tool (*kafka_config*) to configure the instance.

```
kafka_config -a kafka01.mydomain.tld -E -S -i 1 -z zookeeper01.mydomain.tld:2181
```

After this steps a Kafka broker (for either a single instance or a cluster
setup) should be running and configured to start on server boot.

More options can be used on the instance configuration, see the
[Configuration Script](#kafka-configuration-script) section for more details

### Instantiate Zookeeper

Is it possible to use the included Zookeeper installation to instantiate a
Zookeeper node (standalone or as part of a cluster).

In order to end up with a functional Zookeeper node some configurations have to
be performed after instantiating the servers.

To help perform those configurations a small script is included on the AWS
image. The script is called **zookeeper_config**.

#### Zookeeper Configuration Script

The script can and should be used to set some of the Kafka options as well as
setting the Kafka service to start at boot.

```
Usage: zookeeper_config [options]
```

##### Options

* `-D` - Disables the Zookeeper service from start at boot time.
* `-E` - Enables the Zookeeper service to start at boot time.
* `-i <ID>` - Sets the Zookeeper broker ID (default value is '1').
* `-m <MEMORY>` - Sets Zookeeper maximum heap size. Values should be provided following the same Java heap nomenclature.
* `-n <ID:ADDRESS>` - The ID and Address of a cluster node (e.g.: '1:127.0.0.1'). Should be used to set all the Zookeeper nodes. Several Zookeeper nodes can be set by either using extra `-n` options or if separated with a comma on the same `-n` option.
* `-S` - Starts the Zookeeper service after performing the required configurations (if any given).
* `-W <SECONDS>` - Waits the specified amount of seconds before starting the Zookeeper service (default value is '0').

#### Configuring a Zookeeper Node

To prepare an instance to act as a Zookeeper node the following steps need to
be performed.

Run the configuration tool (*zookeeper_config*) to configure the instance.

```
zookeeper_config -E -S
```

After this steps a Zookeeper node (for a standalone setup) should be running
and configured to start on server boot.

For a cluster with more than one Zookeeper node other options have to be
configured on each instance using the same configuration tool
(*zookeeper_config*).

```
zookeeper_config -E -i 1 -n 1:zookeeper01.mydomain.tld -n 2:zookeeper02.mydomain.tld,3:zookeeper03.mydomain.tld -S
```

After this steps, the first node of the Zookeeper cluster (for a three node
cluster) should be running and configured to start on server boot.

More options can be used on the instance configuration, see the
[Configuration Script](#zookeeper-configuration-script) section for more details

## Services

This AMI will have the SSH service running as well as the Kafka services.
The following ports will have to be configured on Security Groups.

| Service      | Port      | Protocol |
|--------------|:---------:|:--------:|
| SSH          | 22        |    TCP   |
| Zookeeper    | 2181      |    TCP   |
| Zookeeper    | 2888:3888 |    TCP   |
| Kafka Broker | 9092      |    TCP   |

## Contributing

1. Fork it!
2. Create your feature branch: `git checkout -b my-new-feature`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin my-new-feature`
5. Submit a pull request

## Versioning

This project uses [SemVer](http://semver.org/) for versioning. For the versions
available, see the [tags on this repository](https://github.com/fscm/packer-templates/tags).

## Authors

* **Frederico Martins** - [fscm](https://github.com/fscm)

See also the list of [contributors](https://github.com/fscm/packer-templates/contributors)
who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE](https://github.com/fscm/packer-templates/LICENSE)
file for details
