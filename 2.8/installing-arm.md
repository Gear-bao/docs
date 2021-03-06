---
layout: default
description: ARM
---
ARM
===

The ArangoDB packages for ARM require the kernel to allow unaligned memory access.
How the kernel handles unaligned memory access is configurable at runtime by
checking and adjusting the contents `/proc/cpu/alignment`.

In order to operate on ARM, ArangoDB requires the bit 1 to be set. This will
make the kernel trap and adjust unaligned memory accesses. If this bit is not
set, the kernel may send a SIGBUS signal to ArangoDB and terminate it.

To set bit 1 in `/proc/cpu/alignment` use the following command as a privileged
user (e.g. root):

    echo "2" > /proc/cpu/alignment

Note that this setting affects all user processes and not just ArangoDB. Setting
the alignment with the above command will also not make the setting permanent,
so it will be lost after a restart of the system. In order to make the setting
permanent, it should be executed during system startup or before starting arangod.

The ArangoDB start/stop scripts do not adjust the alignment setting, but rely on 
the environment to have the correct alignment setting already. The reason for this
is that the alignment settings also affect all other user processes (which ArangoDB
is not aware of) and thus may have side-effects outside of ArangoDB. It is therefore
more reasonable to have the system administrator carry out the changes.

If the alignment settings are not correct, ArangoDB will log an error at startup
and abort.

