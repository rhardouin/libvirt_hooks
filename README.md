libvirt_hooks
=============

Hooks for libvirt

# qemu

Libvirt does not apply traffic shaping on generic ethernet interface for now (it works well for OVS bridges).
For instance, you need to use generic ethernet when using OpenStack and OpenContrail together.

Until there is a fix in libvirt you may want to this hook as a workaround.

## Installation

### Pre requisites

This hook is a Python script so you obviously need Python to be installed on you server (which decent OS doesn't include Python anyway?). It has been tested against Python 2.7 but it should work with Python 2.6.

Note that the default version of the hook uses Unix syslog.


### Let's go

Copy the qemu hook under the following directory on each hypervisor:

    /etc/libvirt/hooks/

Then make it executable by root:

    sudo chmod 744 /etc/libvirt/hooks/qemu

Finally tell libvirt to reload its configuration:

    sudo reload libvirt-bin  # Ubuntu

Or depending on the platform:

    sudo service libvirt-bin reload


## How it works

Whenever an instance is started the hook call tc to apply traffic shaping if necessary.
By necessary I mean if QoS is set on interfaces in the XML instance like so:

```xml
<interface type='ethernet'>
      ...
      <bandwidth>
        <inbound average='500' peak='500' burst='1000'/>
        <outbound average='500' peak='500' burst='1000'/>
      </bandwidth>
      ...
      <target dev='tapa502e991-70'/>
      ...
</interface>
```

## Logging

The hook sends its logs in the Unix syslog. Default level is INFO.

### How to change the log level

Just edit the hook file and change the following line:

```python
LOG.setLevel(logging.INFO)
```

To any valid Python logging level: DEBUG, INFO, WARNING, ERROR Note that the hook does not use the CRITICAL logging level.

### When logging occurs?

**DEBUG** log records are emitted when:

   * the hook is called: command line arguments are logged
   * a tc command is about to be called


**INFO** log records are emitted when:

   * an outbound tc rule is applied
   * an inbound tc rule is applied


**WARNING** log records are emitted when:

   * no bandwidth limitations are found (neither outbound nor inbound). If you use OpenStack this could happens if a Nova flavor has no quota:vif_* extra specs.
     (This is a warning since all production flavors are supposed to be bandwidth limited.)


**ERROR** log records are emitted when:

   * the hook cannot apply the tc rules to a tap.
   * an unexpected error occurs i.e. all but the above error.

### Some examples

    [...] livbirt-hook: WARNING: No bandwidth limitations found in XML: tap=tapce20a28a-2a guest=instance-0001563b

    [...] livbirt-hook:: INFO: tc inbound rules applied: tap=tap74d75d40-4a guest=instance-00015632

    [...] livbirt-hook:: DEBUG: About to execute /sbin/tc qdisc add dev tapf09c54b0-50 ingress
