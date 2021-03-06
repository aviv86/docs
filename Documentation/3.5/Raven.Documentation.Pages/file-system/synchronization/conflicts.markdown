#Conflicts

Each file in the file system has a version number and history (both stored in the file metadata). Every time you modify a file, its version is increased and the old value is added to a history list. 

The conflict detection mechanism works based on these metadata records. A conflict will appear if two files having the same name are synchronized while they have different versions 
and don't have any common records in the history lists. From RavenFS perspective those two files have nothing in common, so they cannot override each other.


##Role of history in synchronization

If a file that has been created on a file system *A* is synchronized to a destination called *B*, then its metadata is sent together with the content.
As a result, the file has the same version and history on the both file systems. Any file modification on *B* server assigns a new version number and sets a source file system identifier in the following metadata:

* `Raven-Synchronization-Version`
* `Raven-Synchronization-Source`

The old ones are incorporated into the history list (each history item is a pair of these values). Now, if the file system *B* wants to synchronize
this file to *A* (*master/master* model), there will be no conflict on *A* because the file version from *A* is contained in the history 
of the file sent by *B*.

{INFO: File relationships}
If the file on a destination file system is an ancestor of the file sent from a source, there is no conflict and the file can be synchronized.
{INFO/}

{WARNING: History length}
RavenFS keeps track of the last 50 file changes. If a file had more than 50 updates since the last synchronization, then a conflict would happen on next
synchronization run (it can happen if a destination server was down for a long time).
{WARNING/}

##Conflict items

If a conflict is detected, an appropriate configuration item is created on a destination file system and stored under the [`Conflicted/[FILENAME]`](./configurations#conflictedfilename) name.
It contains the following info:

* the name of conflicted file,
* the remote file system URL,
* the remote file history,
* the local file history.

The full list of all conflicted files can be retrieved from the `/synchronization/conflicts` endpoint.

##Conflict detection optimization

In general, conflicts can be detected on the destination file system, because existing file metadata is there. However, for the large files it would be a waste to send
a very big file just to determine that it cannot be synchronized because of a conflict. So the detection is performed in a very early in the synchronization
pipeline on a source file system by retrieving remote file metadata and checking it locally. If a conflict is detected, the source creates a conflict item remotely on the destination.

{INFO: Important}
Conflict items are always created and exist on the destination file system.
{INFO/}

##Resolving conflicts

We can resolve a conflict on the destination file system using one of the following strategies:

* `Local (Current) Version` - takes a file that exists on the destination,
* `Remote Version` - takes a file from the source server.

The `Local Version` strategy will incorporate a remote (source) version history into the local (destination) file metadata, so it will look like a file indeed came from a source file system.

If you choose the `Remote Version` then the `Raven-Synchronization-Conflict-Resolution` record will be added to metadata of a file existing on a destination.
It will be used by a source file system during the next synchronization attempt (periodic run or manual execution forced by the user) to determine that it can overwrite a previously conflicted file.

There is also an option to setup default conflict resolution strategy or introduce a custom conflict resolver. You can find out more about dealing with conflicts using Client API [here](../client-api/commands/synchronization/conflicts/resolve).


