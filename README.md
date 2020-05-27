[![Build Status](https://travis-ci.org/intel/sriov-cni.svg?branch=master)](https://travis-ci.org/intel/sriov-cni) [![Go Report Card](https://goreportcard.com/badge/github.com/intel/sriov-cni)](https://goreportcard.com/report/github.com/intel/sriov-cni)

   * [SR-IOV CNI plugin](#sr-iov-cni-plugin)
      * [Build](#build)
      * [Kubernetes Quick Start](#kubernetes-quick-start)
      * [Usage](#usage)
         * [Basic configuration parameters](#basic-configuration-parameters)
         * [Example configurations](#example-configurations)
            * [Kernel driver config](#kernel-driver-config)
            * [Advanced kernel driver config](#advanced-kernel-driver-config)
            * [DPDK userspace driver config](#dpdk-userspace-driver-config)
         * [Advanced configuration](#advanced-configuration)
      * [Contributing](#contributing)

# SR-IOV CNI plugin
NICs with [SR-IOV](http://blog.scottlowe.org/2009/12/02/what-is-sr-iov/) capabilities are managed through physical functions (PFs) and virtual functions (VFs). 

A PF is used by the host and usually represents a single network interface port. VF configurations are applied through the PF. Each VF can be treated as a separate network interface, assigned to a container, and configured with it's own MAC, VLAN and IP, etc.

SR-IOV CNI plugin works with [SR-IOV device plugin](https://github.com/intel/sriov-network-device-plugin) for VF allocation in Kubernetes. A metaplugin such as [Multus](https://github.com/intel/multus-cni) gets the allocated VF's `deviceID`(PCI address) and is responsible for invoking the SR-IOV CNI plugin with that `deviceID`.

## Build

This plugin uses Go modules for dependency management and requires Go 1.12+ to build.

To build the plugin binary:

```
make
```

Upon successful build the plugin binary will be available in `build/sriov`.

## Kubernetes Quick Start
A full guide on orchestrating SR-IOV virtual functions in Kubernetes can be found at the [SR-IOV Device Plugin Repo](https://github.com/intel/sriov-network-device-plugin#quick-start)


To deploy SR-IOV CNI by itself on a Kubernetes 1.16+ cluster:

`kubectl apply -f images/k8s-v1.16/sriov-cni-daemonset.yaml`

**Note** The above deployment is not sufficient to manage and configure SR-IOV virtual functions. [See the full configuration guide for more information.](https://github.com/intel/sriov-network-device-plugin#sr-iov-network-device-plugin)


## Usage
SR-IOV CNI networks are commonly configured using Multus and SR-IOV Device Plugin using Network Attachment Definitions. These 


A Network Attachment Definition for SR-IOV CNI takes the form:

```
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: sriov-net1
  annotations:
    k8s.v1.cni.cncf.io/resourceName: intel.com/intel_sriov_netdevice
spec:
  config: '{
  "type": "sriov",
  "cniVersion": "0.3.1",
  "name": "sriov-network",
  "ipam": {
    "type": "host-local",
    "subnet": "10.56.217.0/24",
    "routes": [{
      "dst": "0.0.0.0/0"
    }],
    "gateway": "10.56.217.1"
  }
}'
```

The `.spec.config` field contains the configuration information used by the SR-IOV CNI.

### Basic configuration parameters 

* `cniVersion` : the version of the CNI spec used.
* `type` : CNI plugin used. "sriov" corresponds to SR-IOV CNI.
* `name` : the name of the network created.
* `ipam` : the configuration of the IP Address Management plugin. Required to designate an IP for a kernel interface.

### Example configurations
The following examples show the config needed to set up basic SR-IOV networking in a container. Each of the json config objects below can be placed in the `.spec.config` field of a Network Attachment Definition to integrate with Multus.

#### Kernel driver config
This is the minimum configuration for a working kernel driver interface using an SR-IOV Virtual Function. It applies an IP address using the host-local ipam plugin in the range of the subnet provided. 

```json
{
  "type": "sriov",
  "cniVersion": "0.3.1",
  "name": "sriov-network",
  "ipam": {
    "type": "host-local",
    "subnet": "10.56.217.0/24",
    "routes": [{
      "dst": "0.0.0.0/0"
    }],
    "gateway": "10.56.217.1"
  }
}
```

#### Advanced kernel driver config
This configuration sets a number of extra parameters that may be key for SR-IOV networks including a vlan tag, disabled spoof checking and enabled trust mode. These parameters are commonly set in more advanced SR-IOV VF based networks.

```json
{
  "cniVersion": "0.3.1",
  "name": "sriov-advanced",
  "type": "sriov",
  "vlan": 1000,
  "spoofchk": "off",
  "trust": "on",
  "ipam": {
    "type": "host-local",
    "subnet": "10.56.217.0/24",
    "routes": [{
      "dst": "0.0.0.0/0"
    }],
    "gateway": "10.56.217.1"
  }
}
```

#### DPDK userspace driver config

The below config will configure a VF using a userspace driver (uio/vfio) for use in a container. If this plugin is used with a VF bound to a dpdk driver then the IPAM configuration will be ignored. Other config parameters should be applicable but implementation may be driver specific. 

```json
{
    "cniVersion": "0.3.1",
    "name": "sriov-dpdk",
    "type": "sriov",
    "vlan": 1000
}
```

### Advanced Configuration 

SR-IOV CNI allows the setting of other SR-IOV parameters such as link state and  Quality of Service. To learn more about how these parameters are set condult the [SR-IOV CNI configuration reference guide](docs/configuration-reference.md)  

## Contributing
To report a bug or request a feature, open an issue on this repo using one of the available templates.
