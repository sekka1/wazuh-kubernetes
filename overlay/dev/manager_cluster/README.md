kustomization
================

For the OSSEC conf files, the only thing that was changed in there from the original
is adding the following to the `<remote>` section to allow the agents to connect.

```
<allowed-ips>100.0.0.0/8</allowed-ips>
<allowed-ips>10.0.0.0/8</allowed-ips>
```
