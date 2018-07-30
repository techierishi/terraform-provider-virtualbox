[![Build Status](https://travis-ci.org/terra-farm/terraform-provider-virtualbox.svg?branch=master)](https://travis-ci.org/terra-farm/terraform-provider-virtualbox)
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fterra-farm%2Fterraform-provider-virtualbox.svg?type=shield)](https://app.fossa.io/projects/git%2Bgithub.com%2Fterra-farm%2Fterraform-provider-virtualbox?ref=badge_shield)

# VirtualBox provider for Terraform

Inspired by [terraform-provider-vix](https://github.com/hooklift/terraform-provider-vix)

Donated to the `terra-farm` group by [`ccll`](https://github.com/ccll)

# How to install

1. go get github.com/terra-farm/terraform-provider-virtualbox

# How to build from source

1. git clone https://github.com/terra-farm/terraform-provider-virtualbox
1. cd terraform-provider-virtualbox
1. dep ensure
1. mv terraform-provider-virtualbox example/
1. cd example/
1. terraform plan
1. terraform apply

# Resources

## "virtualbox_vm"

### Schema

- `name`, string, required: The name of the virtual machine.
- `image`, string, required: The place  of the image file(archive or vagrant box).
- `url`, string, optional, default not set: The url for downloaded vagrant box from external resource (ex. [Ubuntu Vagrant box](https://atlas.hashicorp.com/ubuntu/boxes/trusty64/versions/14.04/providers/virtualbox.box])) . If not set using `image` variable.
- `cpus`, int, optional, default=2: The number of CPUs.
- `memory`, string, optional, default="512mib": The size of memory, allow human friendly units like 'MB', 'MiB'.
- `user_data`, string, optional, default="": User defined data.
- `status`, string, optional, default="running": The status of the VM, allowed values: 'poweroff', 'running'. This value will be updated at runtime to reflect the real status of the VM, and you can also specify it explicitly in config to manually control the status of the VM. This value defaults to 'running', so `terraform apply` will always try to keep the VM running if not specified otherwise.
- `network_adapter`, list: The network adapters in the VM, you can have up to 4 adapters.
  - `.#.type`, string, requried: The type of the network, allowed values: 'nat', 'bridged', 'hostonly', 'internal', 'generic'.
  - `.#.device`, string, optional, default="IntelPro1000MTServer": The model of the virtual hardware device, allowed values: 'PCIII', 'FASTIII', 'IntelPro1000MTDesktop', 'IntelPro1000TServer', 'IntelPro1000MTServer'.
  - `.#.host_interface`, string, optional: Some network type (hostonly, bridged, etc) must bind to a host interface to work properly, use this field to specify the name of the host interface you like to bind to (like 'en0', 'eth1', 'wlan', etc). This should get an improvement, see [TODO](#todo) section below.
  - `.#.status`, string, computed: The status of the network adapter, possible values: 'up', 'down'.
  - `.#.mac_address`, string, computed: The MAC address of the adapter, this is generated by VirtualBox.
  - `.#.ipv4_address`, string, computed: The IPv4 address assigned to the adapter.
  - `.#.ipv4_address_available`, string, computed: Wheather or not an IPv4 address is actaully assigned to the adapter, possible values: "yes", "no".
- `optical_disks`, list: The iso image to attach.

### Network adapter types

- [x] NAT
- [x] bridged

# Example

```hcl
resource "virtualbox_vm" "node" {
    count = 2
    name = "${format("node-%02d", count.index+1)}"

    image = "~/ubuntu-15.04.tar.xz"
    cpus = 2
    memory = "512mib"

    network_adapter {
        type = "nat"
    }

    network_adapter {
        type = "bridged"
        host_interface = "en0"
    }

    optical_disks = ["./cloudinit.iso"]
}

output "IPAddr" {
    # Get the IPv4 address of the bridged adapter (the 2nd one) on 'node-02'
    value = "${element(virtualbox_vm.node.*.network_adapter.1.ipv4_address, 1)}"
}

```

# Limitations

- Experimental provider!

# Example images

- [ubuntu-15.04](https://github.com/ccll/terraform-provider-virtualbox-images/releases/tag/ubuntu-15.04)

- [Ubuntu Vagrant box](https://vagrantcloud.com/ubuntu/boxes/trusty64/versions/20180206.0.0/providers/virtualbox.box)

# TODO

- [x] Optimize resourceVMUpdate(), eliminate unneccessary restarts of VM.
- [x] Auto download image from remote url.
- [ ] Validate downloaded image against checksum.
- [x] Download the same image only once (based on checksum).
- [ ] Re-download corrupted image (based on checksum).


## License
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fterra-farm%2Fterraform-provider-virtualbox.svg?type=large)](https://app.fossa.io/projects/git%2Bgithub.com%2Fterra-farm%2Fterraform-provider-virtualbox?ref=badge_large)
