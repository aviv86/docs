# What are Counters

There are many cases were we would want to add counters to documents.  
For example, say we have a document describing a software release. In addition to tracking the features that are going into the release, 
we also want to count various statistics about the release - how many times a release was downloaded, how many times it was rated, etc.

This seems like it would be very obvious, right? As simple as 1+1. 
However, you need to consider concurrency and distribution. Also, modifying the whole document upon each increment/decrement can be quite expansive.

For such cases RavenDB offers **Distributed Counters**, a *Conflict-free Replicated Data Type* [(CRDT)](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type), 
which is separated from the actual content of the document.

This means that you can increment (or decrement) a value *without modifying the whole document*.  
It also means that RavenDB will be able to automatically handle concurrency on the counters, even when running in a distributed system.
Two separate clients may increment a counter's value at the same time a third user is modifying the parent document.
All of that is handled by RavenDB, the data is updated, distributed across the cluster and the final counter values are tallied.

{NOTE: }
Counters are marked as an **Experimental Feature**. In order to use them you will need to enable experimental features by setting the [Features.Availability]() configuration option.
{NOTE/}

## Basic Operations on Counters

- [Increment]() (or decrement)  
- [Get]() the current value (or Get **Raw-Values** for details)  
- [Delete]()  

More advanced operations include:  

- [Project]() counters in a query   
- [Include]() counters in a **query** and in a **session.Load** operation.
- [Patch]() on counters
- [Track Changes]() in counters

{NOTE: }
You can use counters as a single operation (incrementing a value) or in a batch (incrementing multiple values, or even modifying counters and documents together).  
{NOTE/}

## Relationship between Document and Counter

- Like in [Attachments](../../../client-api/session/attachments/what-are-attachments), counters are linked to a particular document. 
- A counter must reside on an existing document, cannot create on missing document.
- A document can have as many counters as desired.
- A document having counters is marked with a flag: HasCounters 
- A document having counters will hold all it's counter names in the metadata. 
- Modifying a counter does NOT count as a modification to the document.
- Deleting a document will also delete its counters.

## Transactional Guarantees

Whether you used counters as a single operation or in a batch, the operation is transactional and will ensure *full ACIDity*.  
A transaction that includes a counter modification that failed for another reason (for example, concurrency conflict on another document) will revert the counter modification.

## Conflicts

A counter operation is always successful (as long as the document exists, at least) and *can never conflict*.
Even if a document is in conflict mode, counter writes works properly and are NOT affected by merging.

## Revisions and Counters

When the revisions feature is turned on in your database, each counter addition to a document (or deletion from a document) will create a new revision of the document, 
as there will be a change to the document's metadata. 
A revision of a document that has counters will hold in it's metadata all the counter names *and current values* (at the time of revision creation).

{NOTE: }
A counter can only hold values **between \-2147483648 to 2147483647**
{NOTE /}

## Related articles

### Counters

- [Session.CountersFor](../../../client-api/session/counters/counters-for)
- [Low level API](../../../client-api/session/counters/low-level)
- [Query](../../../client-api/session/counters/query)
- [Patch](../../../client-api/session/counters/patch)
- [Include](../../../client-api/session/counters/include)
