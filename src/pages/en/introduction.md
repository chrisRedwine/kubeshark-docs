---
title: The Kubernetes Network Analyzer
description: Real-time K8s network visibility and forensics, capturing and monitoring all traffic and payloads going in, out and across containers, pods, nodes and clusters.
layout: ../../layouts/MainLayout.astro
mascot: Hello
---

**The most efficient way to troubleshoot your K8s apps**

**Kubeshark** is an API Traffic Analyzer for [Kubernetes](https://kubernetes.io/) providing real-time, protocol-level visibility into Kubernetes’ internal network, capturing, dissecting and monitoring all traffic and payloads going in, out and across containers, pods, nodes and clusters.

![Kubeshark UI](/kubeshark-ui.png)

Think [TCPDump](https://en.wikipedia.org/wiki/Tcpdump) and [Wireshark](https://www.wireshark.org/) re-invented for Kubernetes.

## Kubeshark Use-cases

Visit the following sections to read more about use-cases, Kubeshark can be helpful with:
- [Investigation & API Debugging](/en/traffic_investigation)
- [Traffic Recording & Offline Investigation](/en/cloud_forensics)
- [Detection Engineering & Telemetry](/en/actionable_detection)

## Network Analysis
**Kubeshark** uses various packet [capture technologies (e.g. eBPF, PF_RING)](/en/performance#packet-processing-library) and leverages [custom kernel modules](https://en.wikipedia.org/wiki/Loadable_kernel_module) to capture cluster-wide L4 (TCP and UDP) traffic, into distributed PCAP storage and dissect the following application layer protocols:

- [HTTP/1.0](https://datatracker.ietf.org/doc/html/rfc1945)
- [HTTP/1.1](https://datatracker.ietf.org/doc/html/rfc2616)
- [HTTP/2](https://datatracker.ietf.org/doc/html/rfc7540)
- [AMQP](https://www.rabbitmq.com/amqp-0-9-1-reference.html)
- [Apache Kafka](https://kafka.apache.org/protocol)
- [Redis](https://redis.io/topics/protocol)
- [DNS](https://www.iana.org/assignments/dns-parameters/dns-parameters.xhtml)

Kubeshark recognizes [gRPC over HTTP/2](https://grpc.github.io/grpc/core/md_doc__p_r_o_t_o_c_o_l-_h_t_t_p2.html),
[GraphQL over HTTP/1.1](https://graphql.org/learn/serving-over-http/)
and [GraphQL over HTTP/2](https://graphql.org/learn/serving-over-http/).

**Kubeshark** uses [extended BPF (eBPF)](https://en.wikipedia.org/wiki/Berkeley_Packet_Filter) to trace function calls in both the kernel space and the user space.

**Kubeshark** can sniff the [encrypted traffic (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security) in your cluster using
eBPF **without actually doing decryption**. In fact, it hooks into entry and exit points in certain functions inside the
[OpenSSL](https://www.openssl.org/) library and Go's [crypto/tls](https://pkg.go.dev/crypto/tls) package.

**Kubeshark** can recognize service mesh solutions like [Istio](https://istio.io/), [Linkerd](https://linkerd.io/) and other service mesh solutions that use [Envoy Proxy](https://www.envoyproxy.io/) under the hood.

## Actionable Detection Using Scripts & L4/L7 Hooks

With a combination of a [scripting language](/en/automation_scripting), [hooks](/en/automation_hooks), [helpers](/en/automation_helpers) and [jobs](/en/automation_jobs), **Kubeshark** can detect suspicious network behaviors and trigger actions supported by the available integrations (e.g [Slack](/en/integrations_slack), [AWS S3](/en/integrations_aws_s3), [InfluxDB](/en/integrations_influxdb), [Elasticsearch](/en/integrations_elastic) and more).