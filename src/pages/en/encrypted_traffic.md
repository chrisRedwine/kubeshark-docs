---
title: Tracing Encrypted Traffic (TLS)
description: In certain situations, Kubeshark can sniff the encrypted traffic (TLS) in your cluster using eBPF without actually doing decryption.
layout: ../../layouts/MainLayout.astro
mascot:
---

**Using eBPF to sniff encrypted traffic**

Kubeshark can sniff the [encrypted traffic (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security) in your cluster using eBPF **without actually doing decryption**. It hooks into entry and exit points in certain functions inside the [OpenSSL](https://www.openssl.org/) library and Go's [crypto/tls](https://pkg.go.dev/crypto/tls) package.

Kubeshark offers tracing kernel-space and user-space functions using [eBPF](https://prototype-kernel.readthedocs.io/en/latest/bpf/) (Extended Berkeley Packet Filter). eBPF is an in-kernel virtual machine running programs passed from user space. It's first introduced into Linux kernel with version 4.4 and quite matured since then.

TLS payload is marked with an open lock icon to the left of the entry. You can use the helper `tls` as a KFL query to view all the TLS traffic.

![TLS Traffic Example](/tls_traffic.png)

### OpenSSL

Kubeshark attaches [uprobe](https://docs.kernel.org/trace/uprobetracer.html)(s)
to [`SSL_read`](https://www.openssl.org/docs/man1.1.1/man3/SSL_read.html) and [`SSL_write`](https://www.openssl.org/docs/man1.1.1/man3/SSL_write.html) functions to simply reads the incoming unencrypted response and outgoing encrypted request in any TLS/SSL connection.

Languages like Python, Java, PHP, Ruby and Node.js use OpenSSL library for their encryption/decryption work. So, pretty much any program or service that's doing encrypted communication (using TLS) falls into this category.

### Go

Go is a little bit more complicated than OpenSSL but the basic principle is the same.

Go language has two ABIs; **ABI0** and **ABIInternal**. We support both **amd64** and **arm64** so that translates into
a good number of offsets to handle.

We basically probe the [`crypto/tls.(*Conn).Read`](https://github.com/golang/go/blob/go1.17.6/src/crypto/tls/conn.go#L1263) and [`crypto/tls.(*Conn).Write`](https://github.com/golang/go/blob/go1.17.6/src/crypto/tls/conn.go#L1099) just like OpenSLL's `SSL_read` / `SSL_write`. In addition to that we dismantle the targeted Go binaries using Capstone to get the offsets of `ret` mnemonics. Because `uretprobe` does not function properly in Go thanks to its unique ABI.

Lastly, we keep track of the Goroutine ID by using some offsets that we learn by looking at the DWARF table.

### Kernel

We [`kprobe`](https://www.kernel.org/doc/html/latest/trace/kprobes.html) certain tracepoints in the kernel for reasons like doing address resolution (by learning IP and port number for both source and destination) or for matching request-response pairs.

While the methods explained in here sounds a little bit complicated the TLS sniffer has little to no performance impact thanks to the efficient eBPF in-kernel virtual machine of Linux and our carefully written C code. In any case, Linux kernel does not allow injecting a huge number of instructions for probing purposes. So, the kernel itself guarantees no slowdown or crash.

## TLS Capture in Action
<div style="position: relative; padding-bottom: 56.25%; height: 0;"><iframe src="https://www.loom.com/embed/18d9f744402a4b37b1e14c8fd7401aab?sid=0e136344-33af-4739-9899-c41ec0ca0de9" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe></div>

