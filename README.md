# vsock [![Build Status](https://travis-ci.org/mdlayher/vsock.svg?branch=master)](https://travis-ci.org/mdlayher/vsock) [![GoDoc](https://godoc.org/github.com/mdlayher/vsock?status.svg)](https://godoc.org/github.com/mdlayher/vsock) [![Go Report Card](https://goreportcard.com/badge/github.com/mdlayher/vsock)](https://goreportcard.com/report/github.com/mdlayher/vsock)

Package `vsock` provides access to Linux VM sockets (`AF_VSOCK`) for
communication between a hypervisor and its virtual machines.  MIT Licensed.

For more information about VM sockets, check out my blog about
[Linux VM sockets in Go](https://medium.com/@mdlayher/linux-vm-sockets-in-go-ea11768e9e67).

## Go version support

This package supports varying levels of functionality depending on the version
of Go used during compilation. The `net.Listener` and `net.Conn` types produced
by this package are backed by non-blocking I/O, in order to integrate with Go's
runtime network poller in newer versions of Go.

- **Go 1.12.x+ (recommended):**
  - `net.Listener`:
    - `Accept` blocks until a connection is received
    - `Close` can interrupt `Accept` and make it return a permanent error
  - `net.Conn`: full timeout support via `SetDeadline` family of methods
- Go 1.11.x **(not recommended)**:
  - `net.Listener`:
    - `Accept` is **non-blocking** and should be called in a loop, checking for
      `net.Error.Temporary() == true` and sleeping for a short period to avoid
      wasteful, spinning CPU cycles
    - `Close` makes `Accept` return a permanent error on the next
      loop iteration
  - `net.Conn`: full timeout support via `SetDeadline` family of methods
- Go 1.10.x or below: **not supported**

## Stability

At this time, package `vsock` is in a pre-v1.0.0 state. Changes are being made
which may impact the exported API of this package and others in its ecosystem.

**If you depend on this package in your applications, please vendor it or use Go
modules when building your application.**

## Requirements

To make use of VM sockets with QEMU and virtio-vsock, you must have:

- a Linux hypervisor with kernel 4.8+
- a Linux virtual machine on that hypervisor with kernel 4.8+
- QEMU 2.8+ on the hypervisor, running the virtual machine

Before using VM sockets, following modules must be removed on hypervisor:

- `modprobe -r vmw_vsock_vmci_transport`
- `modprobe -r vmw_vsock_virtio_transport_common`
- `modprobe -r vsock`

Once removed, `vhost_vsock` module needs to be enabled on hypervisor:

- `modprobe vhost_vsock`

On VM, you have to enable `vmw_vsock_virtio_transport` module.  This module should automatically load during boot when the vsock device is detected.

To utilize VM sockets, VM needs to be powered on with following `-device` flag:

- `-device vhost-vsock-pci,id=vhost-vsock-pci0,guest-cid=3`

Check out the
[QEMU wiki page on virtio-vsock](http://wiki.qemu-project.org/Features/VirtioVsock)
for more details.  More detail on setting up this environment will be provided
in the future.

## Usage

To try out VM sockets and see an example of how they work, see
[cmd/vscp](https://github.com/mdlayher/vsock/tree/master/cmd/vscp).
This command shows usage of the `vsock.ListenStream` and `vsock.DialStream`
APIs, and allows users to easily test VM sockets on their systems.
