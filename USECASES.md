
## Use cases

### Newly transfering a subvolume from source to destination

With this switch, rsync detects, when a path is a subvolume on the source end, and does not yet exist on the destination end, and instruct the destination to create a subvolume (rather than a regular folder, which would otherwise be the default behavior).

~~~sh
rsync --create-subvolumes <source> <destination>
~~~

The UUID of the source subvolume is preserved in the destination subvolume.

### Converting an existing destination folder to a btrfs subvolume

With this switch, rsync detects, when a path exists and is a regular folder on the destination end, but is a subvolume on the source end, and will attempt to convert the folder to a subvolume before continuing the transfer.

~~~sh
rsync --convert-folders-to-subvolumes <source> <destination>
~~~

This sets the UUID of the subvolume to match the UUID of the source subvolume.

### Converting an existing destination subvolume to a regular folder

With this switch, rsync will detect, when a path exists and is a subvolume on the destination end, but is a regular folder on the source end, and will attempt to convert the subvolume to a folder before continuing the transfer.

~~~sh
rsync --convert-subvolumes-to-folders <source> <destination>
~~~

### Overwrite the UUIDs of existing subvolumes

With this switch, rsync will overwrite the UUIDs of existing subvolumes with the UUIDs of the source subvolumes.

~~~sh
rsync --overwrite-subvolume-uuids <source> <destination>
~~~

Note, that this may break relationships between subvolumes on the destination end and might thus interfere with the function of btrfs send/receive.

### Maintaining the relationships between subvolumes

When transferrings subvolumes between systems, rsync will attempt to maintain the relationships between subvolumes based on their UUIDs,
i.e. when a subvolume has parent or children subvolumes on the source end, rsync will look for subvolumes with the respective UUIDs on the destination end and link them together during transfer.

~~~sh
rsync --maintain-subvolume-relationships <source> <destination>
~~~

In case, linked subvolume UUIDs are not found, rsync will print a warning and continue the transfer.
Since it is possible, that linked subvolumes do not exist on the destination end before rsync creates them in the context of the ongoing transfer, failed subvolume relationsship restoration attempts are retried at the end of the transfer.

