---
title: "Stream Metadata"
section: ".NET API"
version: "4.0.2"
---

Every stream in Event Store has metadata associated with it. Internally, the metadata includes information such as the ACL of the stream and the maximum count and age for the events in the stream. Client code can also put information into stream metadata for use with projections or through the client API.

A common use of this information is to store associated details about an event that is not part of the event. Examples of these are:

-   "which user wrote the event?"
-   "Which application server were they talking to?"
-   "From what IP address did the request come from?"

This type of information is not part of the actual event but is metadata associated with the event. Stream metadata is stored internally as JSON, and you can access it over the HTTP APIs.

## Methods

<!-- TODO: Explanations? -->
<!-- TODO: Moved, check -->
### Reading Stream Metadata

```csharp
Task<StreamMetadataResult> GetStreamMetadataAsync(string stream, UserCredentials userCredentials = null)
```

```csharp
Task<RawStreamMetadataResult> GetStreamMetadataAsRawBytesAsync(string stream, UserCredentials userCredentials = null)
```

### Writing Stream Metadata

```csharp
Task<WriteResult> SetStreamMetadataAsync(string stream, long expectedMetastreamVersion, StreamMetadata metadata, UserCredentials userCredentials = null)
```

```csharp
Task<WriteResult> SetStreamMetadataAsync(string stream, long expectedMetastreamVersion, byte[] metadata, UserCredentials userCredentials = null)
```

## Reading Stream Metadata

To read stream metadata over the .NET API you can use methods found on the `EventStoreConnection`. You can use the `GetStreamMetadata` methods in two ways. The first is to return a fluent interface over the stream metadata, and the second is to return you the raw JSON of the stream metadata.

```csharp
Task<StreamMetadataResult> GetStreamMetadataAsync(string stream, UserCredentials userCredentials = null)
```

This will return a `StreamMetadataResult`. The fields on this result are:

<table>
    <thead>
        <tr>
            <th>Member</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>string Stream</code></td>
            <td>The name of the stream</td>
        </tr>
        <tr>
            <td><code>bool IsStreamDeleted</code></td>
            <td>True is the stream is deleted, false otherwise.</td>
        </tr>
        <tr>
            <td><code>long MetastreamVersion</code></td>
            <td>The version of the metastream format</td>
        </tr>
        <tr>
            <td><code>StreamMetadata Metadata</code></td>
            <td>A StreamMetadata object representing the metadata JSON</td>
        </tr>
    </tbody>
</table>

You can then access the `StreamMetadata` via the `StreamMetadata` object. It contains typed fields for well known stream metadata entries.

<table>
    <thead>
        <tr>
            <th>Member</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>long? MaxAge</code></td>
            <td>The maximum age of events in the stream. Items older than this will be automatically removed.</td>
        </tr>
        <tr>
            <td><code>long? MaxCount</code></td>
            <td>The maximum count of events in the stream. When you have more than count the oldest will be removed.</td>
        </tr>
        <tr>
            <td><code>long? TruncateBefore</code></td>
            <td>When set says that items prior to event 'E' can be truncated and will be removed.</td>
        </tr>
        <tr>
            <td><code>TimeSpan? CacheControl</code></td>
            <td>The head of a feed in the atom api is not cacheable. This allows you to specify a period of time you want it to be cacheable. Low numbers are best here (say 30-60 seconds) and introducing values here will introduce latency over the atom protocol if caching is occuring.</td>
        </tr>
        <tr>
            <td><code>StreamAcl Acl</code></td>
            <td>The access control list for this stream.</td>
        </tr>
    </tbody>
</table>

If instead you want to work with raw JSON you can use the raw methods for stream metadata.

```csharp
Task<RawStreamMetadataResult> GetStreamMetadataAsRawBytesAsync(string stream, UserCredentials userCredentials = null)
```

This will return a `RawStreamMetadataResult`. The fields on this result are:

<table>
    <thead>
        <tr>
            <th>Member</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>string Stream</code></td>
            <td>The name of the stream</td>
        </tr>
        <tr>
            <td><code>bool IsStreamDeleted</code></td>
            <td>True is the stream is deleted, false otherwise.</td>
        </tr>
        <tr>
            <td><code>long MetastreamVersion</code></td>
            <td>The version of the metastream (see <a href="../optimistic-concurrency-and-idempotence">Expected Version</a>)</td>
        </tr>
        <tr>
            <td><code>byte[] Metadata</code></td>
            <td>The raw data of the metadata JSON</td>
        </tr>
    </tbody>
</table>

<span class="note">
If you have security enables, reading metadata may require that you pass credentials. By default it is only allowed for admins though you can change this via default ACLs. If you do not pass credentials and they are required you will receive an `AccessedDeniedException`.
</span>

## Writing Metadata

You can write metadata in both a typed and a raw mechanism. When writing it is generally easier to use the typed mechanism. Both writing mechanisms support an `expectedVersion` which works the same as on any stream and you can use to control concurrency, read [Expected Version](./optimistic-concurrency-and-idempotence) for further details.

```csharp
Task<WriteResult> SetStreamMetadataAsync(string stream, long expectedMetastreamVersion, StreamMetadata metadata, UserCredentials userCredentials = null)
```

The `StreamMetadata` passed here has a builder that you can access via the `StreamMetadata.Create()` method. The options available on the builder are:

<table>
    <thead>
        <tr>
            <th>Method</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>SetMaxCount(long count)</code></td>
            <td>Sets the maximum count of events in the stream.</td>
        </tr>
        <tr>
            <td><code>SetMaxAge(TimeSpan age)</code></td>
            <td>Sets the maximum age of events in the stream.</td>
        </tr>
        <tr>
            <td><code>SetTruncateBefore(long seq)</code></td>
            <td>Sets the event number from which previous events can be scavenged.<</td>
        </tr>
        <tr>
            <td><code>SetCacheControl(TimeSpan cacheControl)</code></td>
            <td>The amount of time for which the stream head is cachable.</td>
        </tr>
        <tr>
            <td><code>SetReadRoles(string[] roles)</code></td>
            <td>Sets the roles that are allowed to read the underlying stream.</td>
        </tr>
        <tr>
            <td><code>SetWriteRoles(string[] roles)</code></td>
            <td>Sets the roles that are allowed to write to the underlying stream.</td>
        </tr>
        <tr>
            <td><code>SetDeleteRoles(string[] roles)</code></td>
            <td>Sets the roles that are allowed to delete the underlying stream.</td>
        </tr>
        <tr>
            <td><code>SetMetadataReadRoles(string[] roles)</code></td>
            <td>Sets the roles that are allowed to read the metadata stream.</td>
        </tr>
        <tr>
            <td><code>SetMetadataWriteRoles(string[] roles)</code></td>
            <td>Sets the roles that are allowed to write the metadata stream. Be careful with this privilege as it gives all of the privileges for a stream as that use can assign themselves any other privilege.</td>
        </tr>
        <tr>
            <td><code>SetCustomMetadata(string key, string value)</code></td>
            <td>The SetCustomMetadata method and overloads allow the setting of arbitrary custom fields into the stream metadata.</td>
        </tr>
    </tbody>
</table>

You can add user-specified metadata via the `SetCustomMetadata` overloads. Some examples of good uses of user-specified metadata are:

-   which adapter is responsible for populating a stream.
-   which projection caused a stream to be created.
-   a correlation ID of some business process.

```csharp
Task<WriteResult> SetStreamMetadataAsync(string stream, long expectedMetastreamVersion, byte[] metadata, UserCredentials userCredentials = null)
```

This method will put the data that is in metadata as the stream metadata. Metadata in this case can be anything in a vector of bytes however the server only understands JSON. Read [Access Control Lists](server/access-control-lists) for more information on the format in JSON for access control lists.

<span class="note">
Writing metadata may require that you pass credentials if you have security enabled by default it is only allowed for admins though you can change this via default ACLs. If you do not pass credentials and they are required you will receive an `AccessedDeniedException`.
</span>
