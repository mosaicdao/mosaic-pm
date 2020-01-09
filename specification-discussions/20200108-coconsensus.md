---
title: 'Coconsensus'
disqus:
---

Mosaic Implementation Discussion: Coconsensus
===

| version | Last updated | Component          |
| ------- | ------------ | ------------------ |
| 0.14    | 08/01/2020    | Coconsensus contract |


Editor: Benjamin Bollen

## Overview

Coconsensus mirrors the consensus contract on the auxiliary chain;
however, notable differences make it worthwhile to describe it in its own right.

Coconsensus can mirror the metachains storage from consensus, but it should not track the metablocks, rather it can track the checkpoints which have a state machine 

```
    enum CheckpointCommitmentStatus {
        Undefined,
        Finalised,
        Committed
    }
```

- update validators

---
## Meeting notes
### Meeting 1
date/time:
attendees:
