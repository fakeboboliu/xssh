# free golang/x/crypto/ssh

> You are free to be insecure

This repo contains a fork of [golang.org/x/crypto/ssh](golang.org/x/crypto@v0.28.0/ssh) (currently from 0.28.0).

With UMAC support using [fakeboboliu/umac](https://github.com/fakeboboliu/umac).

It's a drop-in replacement for the original package, with the same API.

The patch commits are separated, so you can do it manually and review what to do.

## Why

Using UMAC is less secure than the original MACs, but it's much faster,
especially on platforms that mean to be slow (e.g. A VPS with 1vCPU).

## How to use

```go
package main

import (
	ssh "github.com/fakeboboliu/xssh"
	// instead of "golang.org/x/crypto/ssh"
)

func main() {
	config := &ssh.ServerConfig{
		Config: ssh.Config{
			Ciphers: []string{"aes128-ctr"},
			MACs:    []string{"umac-128@openssh.com"}, // set or not, UMACs are already the default list
		},
	}
	// use ssh as usual
}
```

## Benchmark

test on AWS EC2 t3.nano(2 vCPU) with full credits (actually a lightsail instance, but they are basically the same)

```
goos: linux
goarch: amd64
pkg: github.com/fakeboboliu/umac
cpu: Intel(R) Xeon(R) Platinum 8259CL CPU @ 2.50GHz
BenchmarkHMACSHA256_1K
BenchmarkHMACSHA256_1K-2   	 1428729	      3614 ns/op	 283.34 MB/s
BenchmarkHMACSHA256_32
BenchmarkHMACSHA256_32-2   	10101997	       670.2 ns/op	  47.75 MB/s
BenchmarkHMACMD5_1K
BenchmarkHMACMD5_1K-2      	 2584860	      2161 ns/op	 473.94 MB/s
BenchmarkHMACMD5_32
BenchmarkHMACMD5_32-2      	14015890	       415.3 ns/op	  77.05 MB/s
BenchmarkUMAC64_1K
BenchmarkUMAC64_1K-2      	14754679	       379.2 ns/op	2700.67 MB/s
BenchmarkUMAC64_32
BenchmarkUMAC64_32-2       	42769585	       152.1 ns/op	 210.40 MB/s
BenchmarkUMAC128_1K
BenchmarkUMAC128_1K-2      	10143998	       595.2 ns/op	1720.37 MB/s
BenchmarkUMAC128_32
BenchmarkUMAC128_32-2      	36241742	       172.0 ns/op	 186.00 MB/s
```

It's actually pretty powerful, consider a lot of Xeon E5v2 still on duty.

## See Also

- [Making the fastest SSH connection](https://blog.twogate.com/entry/2020/07/30/benchmarking-ssh-connection-what-is-the-fastest-cipher)
- Tip: The fastest transferring files with ssh: `tar -c <src>|pv|zstd -9 -T0|ssh -c aes128-ctr -o "MACs umac-64@openssh.com" <addr> "zstd -d | tar -xC <dst>"`