## Configuration refence - SR-IOV CNI

The SR-IOV CNI configures networks through a CNI spec configuration object. In a Kubernetes cluster set up with Multus this object is most often delivered as a Network Attachment Definition. 


### Parameters
* `name` (string, required): the name of the network
* `type` (string, required): "sriov"
* `ipam` (dictionary, optional): IPAM configuration to be used for this network.
* `deviceID` (string, required): A valid pci address of an SRIOV NIC's VF. e.g. "0000:03:02.3"
* `vlan` (int, optional): VLAN ID to assign for the VF. Value must be in the range 0-4094 (0 for disabled, 1-4094 for valid VLAN IDs).
* `vlanQoS` (int, optional): VLAN QoS to assign for the VF. Value must be in the range 0-7. This option requires `vlan` field to be set to a non-zero value. Otherwise, the error will be returned.
* `mac` (string, optional): MAC address to assign for the VF
* `spoofchk` (string, optional): turn packet spoof checking on or off for the VF
* `trust` (string, optional): turn trust setting on or off for the VF
* `link_state` (string, optional): enforce link state for the VF. Allowed values: auto, enable, disable. Note that driver support may differ for this feature. For example, `i40e` is known to work but `igb` doesn't.
* `min_tx_rate` (int, optional): change the allowed minimum transmit bandwidth, in Mbps, for the VF. Setting this to 0 disables rate limiting. The min_tx_rate value should be <= max_tx_rate. Support of this feature depends on NICs and drivers.
* `max_tx_rate` (int, optional): change the allowed maximum transmit bandwidth, in Mbps, for the VF.
Setting this to 0 disables rate limiting.

