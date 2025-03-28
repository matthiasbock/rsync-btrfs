
# rsync-btrfs

## Btrfs Send/Receive and Identifier Preservation

When using btrfs send/receive to transfer subvolumes between systems:

Subvolume IDs:
* NOT preserved during transfer
* The receiving filesystem assigns new, local subvolume IDs
* This is by design since subvolume IDs are filesystem-local identifiers

UUIDs:
* ARE preserved during transfer
* The UUID (Universally Unique Identifier) is maintained across the send/receive operation
* This makes UUIDs useful for tracking the same logical subvolume across different systems

Other Preserved Properties:
* Creation timestamps
* Subvolume flags and attributes
* Internal parent-child relationships between snapshots
* File content, permissions, ownership, and extended attributes

## Important Considerations

Kernel Support:
* Requires a relatively recent Linux kernel (5.7+)
* Requires recent btrfs-progs (5.7+)

Root Privileges:
* Changing UUIDs requires root/sudo access

Implications:
* Changing UUIDs can break tools that track subvolumes by UUID
* Can disrupt incremental send/receive operations
* May affect backup systems that rely on UUID consistency

Use Cases:
* Resolving UUID conflicts when merging filesystems
* Setting up mirrored systems with identical UUIDs
* Testing and development scenarios

## UUID-Based Subvolume Identification

Sender identifies files as belonging to specific UUID-identified subvolumes:
* Receiver recognizes these UUIDs and maps them to local subvolumes
* Extent Deduplication Across Transfers

Instead of transferring full file content, the system identifies:
* Which extents already exist in the destination subvolume
* Which extents exist elsewhere on the destination filesystem
* Which extents must be transferred completely

## Metadata-Only Updates

For files where extents already exist:
* Only transfer metadata changes
* Reuse existing data blocks through btrfs's native CoW mechanisms

## Technical Challenges

Implementing this would involve solving several complex problems:

Extent Fingerprinting:
* Need an efficient way to identify and compare extents across systems
* Could use cryptographic hashes of extent content
* Challenge: Determining optimal extent boundaries

Extent Mapping Protocol:
* Sender and receiver need to exchange extent metadata efficiently
* Protocol must handle extent verification and fallback mechanisms
* Must work within or alongside rsync's existing delta-transfer algorithm

Filesystem Integration:
* Requires deeper integration with btrfs than typical userspace tools
* May need custom kernel modules or filesystem extensions
* Needs to handle btrfs's internal extent reference counting

Performance Considerations:
* Computing and comparing extent fingerprints adds overhead
* Benefit depends on extent reuse probability
* Need to balance metadata exchange vs. just sending the data
