# AWS Security Group exporter for Prometheus

**[Motivation](#motivation)** |
**[How does it work](#how-does-it-work)** |
**[Requirements](#requirements)** |
**[Docker image settings](#docker-image-settings)** |
**[Examples](#examples)** |
**[Grafana setup](#grafana-setup)**

[![Build Status](https://travis-ci.com/cytopia/aws-ec2-sg-exporter.svg?branch=master)](https://travis-ci.com/cytopia/aws-ec2-sg-exporter)
[![Tag](https://img.shields.io/github/tag/cytopia/aws-ec2-sg-exporter.svg)](https://github.com/cytopia/aws-ec2-sg-exporter/releases)
[![](https://images.microbadger.com/badges/version/cytopia/aws-ec2-sg-exporter:latest.svg?&kill_cache=1)](https://microbadger.com/images/cytopia/aws-ec2-sg-exporter:latest "aws-ec2-sg-exporter")
[![](https://images.microbadger.com/badges/image/cytopia/aws-ec2-sg-exporter:latest.svg?&kill_cache=1)](https://microbadger.com/images/cytopia/aws-ec2-sg-exporter:latest "aws-ec2-sg-exporter")
[![](https://img.shields.io/docker/pulls/cytopia/aws-ec2-sg-exporter.svg)](https://hub.docker.com/r/cytopia/aws-ec2-sg-exporter)
[![](https://img.shields.io/badge/github-cytopia%2Faws--ec2--sg--exporter-red.svg)](https://github.com/cytopia/aws-ec2-sg-exporter "github.com/cytopia/aws-ec2-sg-exporter")
[![License](https://img.shields.io/badge/license-MIT-%233DA639.svg)](https://opensource.org/licenses/MIT)

![Grafana](doc/grafana-graph.png "Grafana Graph Example")

A dockerized Prometheus exporter that compares desired/wanted IPv4/IPv6 CIDR against currently applied inbound CIDR rules in your AWS security group(s).

[![Docker hub](http://dockeri.co/image/cytopia/aws-ec2-sg-exporter?&kill_cache=1)](https://hub.docker.com/r/cytopia/aws-ec2-sg-exporter)


## Motivation

Some IP addresses such as Cloudfront edge nodes change frequently and you want to make sure that those are always inbound allowed in your security groups. This exporter does exactly this and can easily be hooked
up with Alertmanager to trigger alerts in case you get out of sync.

## How does it work

#### Desired/Wanted IP address CIDR

You have to provide a command, which is parsable by `eval` and will evalute to your desired/wanted IP address CIDR. As an example:
```bash
eval "dig +short nat.travisci.net | xargs -n1 -I% echo \"%/32\""
```
#### Applied security Group CIDR

You have to provide the following:

* Security group name
* AWS region where the security group resides
* Security group rule protocol (e.g.: `tcp`, `udp`, `icmp`, ...)
* Security group rule from port (e.g.: `80`, `443`, ...)

#### Output

The exporter will then output Prometheus readable information as such:
```bash
# HELP aws_ec2_sg_compare Determines If CIDR is applied to security group.
# TYPE aws_ec2_sg_compare counter
aws_ec2_sg_compare{name="sg-name",region="us-east-1",proto="tcp",from_port="80",ip="v4",cidr="104.154.113.151/32"} 0
aws_ec2_sg_compare{name="sg-name",region="us-east-1",proto="tcp",from_port="80",ip="v4",cidr="104.154.120.187/32"} 1
aws_ec2_sg_compare{name="sg-name",region="us-east-1",proto="tcp",from_port="80",ip="v4",cidr="104.198.131.58/32"} 1
aws_ec2_sg_compare{name="sg-name",region="us-east-1",proto="tcp",from_port="80",ip="v4",cidr="207.254.16.35/32"} 1
aws_ec2_sg_compare{name="sg-name",region="us-east-1",proto="tcp",from_port="80",ip="v4",cidr="207.254.16.36/32"} 1
aws_ec2_sg_compare{name="sg-name",region="us-east-1",proto="tcp",from_port="80",ip="v4",cidr="207.254.16.38/32"} 1
aws_ec2_sg_compare{name="sg-name",region="us-east-1",proto="tcp",from_port="80",ip="v4",cidr="207.254.16.39/32"} 1
```
1. A value of `1` means the desired/wanted IP CIDR is applied to the security group
2. A value of `0` means the desired/wanted IP CIDR is not applied to the security group

## Requirements

You will need AWS access key and secret with the following permission:
```
ec2:DescribeSecurityGroups
```

## Docker image settings

### Environment variables

You can specify up to 4 security group checks: `SG1_*`, `SG2_*`, `SG3_*` and `SG4_*`.

| Variable                | Description |
|-------------------------|-------------|
| `AWS_ACCESS_KEY_ID`     | The AWS access key (required to connect to AWS to check the sg rules) |
| `AWS_SECRET_ACCESS_KEY` | The AWS secret key (required to connect to AWS to check the sg rules) |
| `AWS_SESSION_TOKEN`     | (Optional) The AWS session token |
| | |
| `UPDATE_TIME`           | Time interval in sec for how often to update metrics (default: `60`) |
| | |
| `SG1_NAME`              | Name of the security group on AWS |
| `SG1_REGION`            | Region the security group resides in |
| `SG1_PROTO`             | Security group rule protocol: `tcp`, `udp`, `icmp` or a protocol number |
| `SG1_FROM_PORT`         | Security group rule from port |
| `SG1_IP4_CMD`           | The command that evaluates to newline-separated IPv4 IP address CIDR <strong><sup>[1]</sup></strong> |
| `SG1_IP6_CMD`           | The command that evaluates to newline-separated IPv6 IP address CDIR <strong><sup>[1]</sup></strong> |
| | |
| `SG2_NAME`              | Name of the security group on AWS |
| `SG2_REGION`            | Region the security group resides in |
| `SG2_PROTO`             | Security group rule protocol: `tcp`, `udp`, `icmp` or a protocol number |
| `SG2_FROM_PORT`         | Security group rule from port |
| `SG2_IP4_CMD`           | The command that evaluates to newline-separated IPv4 IP address CIDR <strong><sup>[1]</sup></strong> |
| `SG2_IP6_CMD`           | The command that evaluates to newline-separated IPv6 IP address CDIR <strong><sup>[1]</sup></strong> |
| | |
| `SG3_NAME`              | Name of the security group on AWS |
| `SG3_REGION`            | Region the security group resides in |
| `SG3_PROTO`             | Security group rule protocol: `tcp`, `udp`, `icmp` or a protocol number |
| `SG3_FROM_PORT`         | Security group rule from port |
| `SG3_IP4_CMD`           | The command that evaluates to newline-separated IPv4 IP address CIDR <strong><sup>[1]</sup></strong> |
| `SG3_IP6_CMD`           | The command that evaluates to newline-separated IPv6 IP address CDIR <strong><sup>[1]</sup></strong> |
| | |
| `SG4_NAME`              | Name of the security group on AWS |
| `SG4_REGION`            | Region the security group resides in |
| `SG4_PROTO`             | Security group rule protocol: `tcp`, `udp`, `icmp` or a protocol number |
| `SG4_FROM_PORT`         | Security group rule from port |
| `SG4_IP4_CMD`           | The command that evaluates to newline-separated IPv4 IP address CIDR <strong><sup>[1]</sup></strong> |
| `SG4_IP6_CMD`           | The command that evaluates to newline-separated IPv6 IP address CDIR <strong><sup>[1]</sup></strong> |

**[1]**: `SG*_IP4_CMD` and `SG*_IP6_CMD` are mutually exclusive. Also note that evaluated
IP address CIDR are only checked against security group rules that match the protocol (`SG*_PROTO`)
and also match the from port (`SG*_FROM_PORT`).


### Mount points

**None**


### Exposed ports

| External  | Internal | Description |
|-----------|----------|-------------|
| Up to you | `8080`   | Where the `aws-ec2-sg-exporter` provides metrics via HTTP |


## Metrics

This exporter outputs metrics in the following format:
```bash
# HELP aws_ec2_sg_compare Determines If CIDR is applied to security group.
# TYPE aws_ec2_sg_compare counter
aws_ec2_sg_compare{name="sg-name",region="us-east-1",proto="tcp",from_port="443",ip="v4",cidr="10.4.1.1/32",sg_id="sg-xxxxx",errno="0",error=""} 1
```
The following table describes each of the key/value paris:

| Key         | Value |
|-------------|-------|
| `name`      | The security group name as specified by `SG*_NAME` |
| `region`    | The security group region as specified by `SG*_REGION` |
| `proto`     | The security group rule protocol as specified by `SG*_PROTO` |
| `from_port` | The security group rule from port as specified by `SG*_FROM_PORT` |
| `ip`        | IP version of desired/wanted CIDR to be available in your security group by `proto` and `from_port` |
| `cidr`      | The desired/wanted IP to be available in your security group by `proto` and `from_port` |
| `sg_id`     | The security group ID found by `name` and `region`. If this is empty then either zero or more multiple security groups were found. |
| `errno`     | 0: One security group was found (OK)<br/>1: No security group was found (ERR)<br/>2: Multiple security groups were found (ERR) |
| `error`     | The corresponding error message for `errno` |

* A value of `1` means the desired/wanted IP CIDR is applied to the security group
* A value of `0` means the desired/wanted IP CIDR is not applied to the security group


## Examples

### Scenario 1 - Travis
Check if your security group named `my-sg` (in us-east-1) allows all inbound IPv4 addresses from Travis-CI via `tcp` on `https`.

#### Desired/wanted IP CIDR
Ensure you have a working command which can be interpretated by `eval` and that outputs CIDR (with `/[0-9]+` appended) of your desired ranges:
```bash
$ eval "dig +short nat.travisci.net | xargs -n1 -I% echo \"%/32\""
```
```
35.184.226.236/32
35.188.1.99/32
35.188.73.34/32
35.192.85.2/32
35.192.136.167/32
...
```

#### Run `aws-ec2-sg-exporter`
```bash
docker run -it --rm \
	-p 9000:8080 \
	\
	-e AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID} \
	-e AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY} \
	-e AWS_SESSION_TOKEN=${AWS_SESSION_TOKEN} \
	\
	-e SG1_NAME="my-sg" \
	-e SG1_REGION="us-east-1" \
	-e SG1_PROTO="tcp" \
	-e SG1_FROM_PORT="443" \
	-e SG1_IP4_CMD="dig +short nat.travisci.net | xargs -n1 -I% echo \"%/32\"" \
	cytopia/aws-ec2-sg-exporter
```

#### Check output
Check the output via curl:
```bash
$ curl localhost:9000`
```
```bash
# HELP aws_ec2_sg_compare Determines If CIDR is applied to security group.
# TYPE aws_ec2_sg_compare counter
aws_ec2_sg_compare{name="my-sg",region="us-east-1",proto="tcp",from_port="443",ip="v4",cidr="35.184.226.236/32"} 1
aws_ec2_sg_compare{name="my-sg",region="us-east-1",proto="tcp",from_port="443",ip="v4",cidr="35.188.1.99/32"} 1
aws_ec2_sg_compare{name="my-sg",region="us-east-1",proto="tcp",from_port="443",ip="v4",cidr="35.188.73.34/32"} 1
aws_ec2_sg_compare{name="my-sg",region="us-east-1",proto="tcp",from_port="443",ip="v4",cidr="35.192.85.2/32"} 1
aws_ec2_sg_compare{name="my-sg",region="us-east-1",proto="tcp",from_port="443",ip="v4",cidr="35.192.136.167/32"} 0
...
```

As you can see, the last line returns a `0`, which means this IP CIDR is missing in your security group.


### Scenario 2 - Cloudfront

* Check if your security group named `my-sg4` (in us-east-1) allows all inbound IPv4 addresses from Cloudfront edge-nodes via `tcp` on `https`.
* Check if your security group named `my-sg6` (in us-east-1) allows all inbound IPv6 addresses from Cloudfront edge-nodes via `tcp` on `https`.

#### Desired/wanted IP CIDR
Ensure you have a working command which can be interpretated by `eval` and that outputs CIDR (with `/[0-9]+` appended) of your desired ranges:
```bash
$ eval "curl -sS https://ip-ranges.amazonaws.com/ip-ranges.json \
	| jq -r '.prefixes \
		| sort_by(.ip_prefix)[] \
		| select( .service | contains(\"CLOUDFRONT\")) \
		| select ( .region | test(\"^(GLOBAL|us-|eu-)\")) \
		| .ip_prefix'"
```
```
13.224.0.0/14
13.249.0.0/16
13.32.0.0/15
13.35.0.0/16
13.52.204.0/23
...
```
```bash
$ eval "curl -sS https://ip-ranges.amazonaws.com/ip-ranges.json \
	| jq -r '.ipv6_prefixes \
		| sort_by(.ipv6_prefixes)[] \
		| select( .service | contains(\"CLOUDFRONT\")) \
		| select ( .region | test(\"^(GLOBAL|us-|eu-)\")) \
		| .ipv6_prefix'"
```
```
2600:9000:eee::/48
2600:9000:4000::/36
2600:9000:3000::/36
2600:9000:f000::/36
2600:9000:fff::/48
...
```

#### Run `aws-ec2-sg-exporter`
```bash
docker run -it --rm \
	-p 9000:8080 \
	\
	-e AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID} \
	-e AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY} \
	-e AWS_SESSION_TOKEN=${AWS_SESSION_TOKEN} \
	\
	-e SG1_NAME="my-sg4" \
	-e SG1_REGION="us-east-1" \
	-e SG1_PROTO="tcp" \
	-e SG1_FROM_PORT="443" \
	-e SG1_IP4_CMD="curl -sS https://ip-ranges.amazonaws.com/ip-ranges.json | jq -r '.prefixes | sort_by(.ip_prefix)[] | select( .service | contains(\"CLOUDFRONT\")) | select ( .region | test(\"^(GLOBAL|us-|eu-)\")) | .ip_prefix'" \
	\
	-e SG2_NAME="my-sg6" \
	-e SG2_REGION="us-east-1" \
	-e SG2_PROTO="tcp" \
	-e SG2_FROM_PORT="443" \
	-e SG2_IP6_CMD="curl -sS https://ip-ranges.amazonaws.com/ip-ranges.json | jq -r '.ipv6_prefixes | sort_by(.ipv6_prefixes)[] | select( .service | contains(\"CLOUDFRONT\")) | select ( .region | test(\"^(GLOBAL|us-|eu-)\")) | .ipv6_prefix'" \
	cytopia/aws-ec2-sg-exporter
```

#### Check output
Check the output via curl:
```bash
$ curl localhost:9000`
```
```bash
# HELP aws_ec2_sg_compare Determines If CIDR is applied to security group.
# TYPE aws_ec2_sg_compare counter
aws_ec2_sg_compare{name="my-sg4",region="us-east-1",proto="tcp",from_port="443",ip="v4",cidr="13.224.0.0/14"} 1
aws_ec2_sg_compare{name="my-sg4",region="us-east-1",proto="tcp",from_port="443",ip="v4",cidr="13.249.0.0/16"} 0
aws_ec2_sg_compare{name="my-sg4",region="us-east-1",proto="tcp",from_port="443",ip="v4",cidr="13.32.0.0/15"} 1
aws_ec2_sg_compare{name="my-sg4",region="us-east-1",proto="tcp",from_port="443",ip="v4",cidr="13.35.0.0/16"} 1
aws_ec2_sg_compare{name="my-sg4",region="us-east-1",proto="tcp",from_port="443",ip="v4",cidr="13.52.204.0/23"} 1
...
aws_ec2_sg_compare{name="my-sg6",region="us-east-1",proto="tcp",from_port="443",ip="v6",cidr="2600:9000:eee::/48"} 1
aws_ec2_sg_compare{name="my-sg6",region="us-east-1",proto="tcp",from_port="443",ip="v6",cidr="2600:9000:4000::/36"} 1
aws_ec2_sg_compare{name="my-sg6",region="us-east-1",proto="tcp",from_port="443",ip="v6",cidr="2600:9000:3000::/36"} 1
aws_ec2_sg_compare{name="my-sg6",region="us-east-1",proto="tcp",from_port="443",ip="v6",cidr="2600:9000:f000::/36"} 1
aws_ec2_sg_compare{name="my-sg6",region="us-east-1",proto="tcp",from_port="443",ip="v6",cidr="2600:9000:fff::/48"} 0
...
```

As you can see, the second line ipv4 address returns a `0` and the last ipv6 address returns a `0`, which means these IP CIDR are missing in your security groups.


## Grafana setup

* Align the `Min time interval` with what you have set `UPDATE_TIME` to.
* Add you metrics by the name of your specified security group name
* Set the legend to `{{ cidr }}` to have only the CIDR displayed

![Grafana](doc/grafana-setup.png "Grafana Setup Example")

Once this is done, your graph will look similar to this one:

![Grafana](doc/grafana-graph.png "Grafana Graph Example")


## License

**[MIT License](LICENSE)**

Copyright (c) 2019 [cytopia](https://github.com/cytopia)
