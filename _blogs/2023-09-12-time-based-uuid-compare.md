---
title: "How to compare time based UUID in Java"
author: yiqi
permalink: /blogs/2023-09-12-time-based-uuid-compare
date: 2023-09-12
collection: blogs
tag:
- Programming
- Java
---

## UUID.compareTo?

A common mistake is to directly compare the UUID itself using ```UUID.compareTo```. Though it looks very natural and correct, it's actually the wrong way to compare time based (version 1) UUID.  

In short, one (time based) UUID being greater (less) than the other DOES NOT necessary mean the represented timestamp being greater (less). This is due to the generation of time based UUID does not guarantee that the first half of the UUID bits represent the time.  

If you are interested on how the generation works, read the following:  
* [UUID Versions Explained](https://www.uuidtools.com/uuid-versions-explained)
* [wikipedia](https://en.wikipedia.org/wiki/Universally_unique_identifier)

## Extracting timestamp from UUID

The correct way to compare time based UUID is to extract and compare the timestamp.  

```java
import java.time.Instant;
import java.util.UUID;

public class UUIDUtil {

    private static final long NUM_HUNDRED_NANOS_IN_A_SECOND = 10_000_000L;

    private static final long NUM_HUNDRED_NANOS_FROM_UUID_EPOCH_TO_UNIX_EPOCH = 122_192_928_000_000_000L;


    /**
     * Extracts the Instant (with the maximum available 100ns precision) from the given time-based (version 1) UUID.
     *
     * @return the {@link Instant} extracted from the given time-based UUID
     * @throws UnsupportedOperationException If this UUID is not a version 1 UUID
     */
    public static Instant getInstantFromUUID(final UUID uuid) {
        final long hundredNanosSinceUnixEpoch = uuid.timestamp() - NUM_HUNDRED_NANOS_FROM_UUID_EPOCH_TO_UNIX_EPOCH;
        final long secondsSinceUnixEpoch = hundredNanosSinceUnixEpoch / NUM_HUNDRED_NANOS_IN_A_SECOND;
        final long nanoAdjustment = ((hundredNanosSinceUnixEpoch % NUM_HUNDRED_NANOS_IN_A_SECOND) * 100);
        return Instant.ofEpochSecond(secondsSinceUnixEpoch, nanoAdjustment);
    }
}
```

## How Cassandra handles it

The good news is, Cassandra implements the right way to compare UUID. Say you have a schema where some field is a time based UUID, you can safely use it for ordering so that the data is order by some timestamp you defined.  

```cql
CREATE TABLE IF NOT EXISTS message_snapshot
(
    author      text,
    snapshot_id uuid,
    status      text,
    updated     timestamp,

    PRIMARY KEY ((author), snapshot_id)
) WITH CLUSTERING ORDER BY (snapshot_id desc);
```