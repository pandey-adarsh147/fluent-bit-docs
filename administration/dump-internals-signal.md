# Dump Internals / Signal

When the service is running we can export [metrics](monitoring.md) to see the overall status of the data flow of the service. But there are other use cases where we would like to know the current status of the internals of the service, specifically to answer questions like _what's the current status of the internal buffers ?_ , the Dump Internals feature is the answer.

Fluent Bit v1.4 introduces the Dump Internals feature that can be triggered easily from the command line triggering the `CONT` Unix signal.

{% hint style="info" %}
note: this feature is only available on Linux and BSD family operating systems
{% endhint %}

## Usage

Run the following `kill` command to signal Fluent Bit:

```text
kill -CONT `pidof fluent-bit`
```

> The command `pidof` aims to lookup the Process ID of Fluent Bit. You can replace the

Fluent Bit will dump the following information to the standard output interface \(stdout\):

```text
[engine] caught signal (SIGCONT)
[2020/03/23 17:39:02] Fluent Bit Dump

===== Input =====
syslog_debug (syslog)
│
├─ status
│  └─ overlimit     : no
│     ├─ mem size   : 60.8M (63752145 bytes)
│     └─ mem limit  : 61.0M (64000000 bytes)
│
├─ tasks
│  ├─ total tasks   : 92
│  ├─ new           : 0
│  ├─ running       : 92
│  └─ size          : 171.1M (179391504 bytes)
│
└─ chunks
   └─ total chunks  : 92
      ├─ up chunks  : 35
      ├─ down chunks: 57
      └─ busy chunks: 92
         ├─ size    : 60.8M (63752145 bytes)
         └─ size err: 0

===== Storage Layer =====
total chunks     : 92
├─ mem chunks    : 0
└─ fs chunks     : 92
   ├─ up         : 35
   └─ down       : 57
```

## Input Plugins Dump

The dump provides insights for every input instance configured.

### Status

Overall ingestion status of the plugin.

| Entry | Sub-entry | Description |
| :--- | :--- | :--- |
| overlimit |  | If the plugin has been configured with [Mem\_Buf\_Limit](backpressure.md), this entry will report if the plugin is over the limit or not at the moment of the dump. If it is overlimit, it will print `yes`, otherwise `no`. |
|  | mem\_size | Current memory size in use by the input plugin in-memory. |
|  | mem\_limit | Limit set by Mem\_Buf\_Limit. |

### Tasks

When an input plugin ingest data into the engine, a Chunk is created. A Chunk can contains multiple records. Upon flush time, the engine creates a Task that contains the routes for the Chunk associated in question.

The Task dump describes the tasks associated to the input plugin:

| Entry | Description |
| :--- | :--- |
| total\_tasks | Total number of active tasks associated to data generated by the input plugin. |
| new | Number of tasks not assigned yet to an output plugin. Tasks are in `new` status for a very short period of time \(most of the time this value is very low or zero\). |
| running | Number of active tasks being processed by output plugins. |
| size | Amount of memory used by the Chunks being processed \(Total chunks size\). |

### Chunks

The Chunks dump tells more details about all the chunks that the input plugin has generated and are still being processed.

Depending of the buffering strategy and limits imposed by configuration, some Chunks might be `up` \(in memory\) or `down` \(filesystem\).

| Entry | Sub-entry | Description |
| :--- | :--- | :--- |
| total\_chunks |  | Total number of Chunks generated by the input plugin that are still being processed by the engine. |
| up\_chunks |  | Total number of Chunks that are loaded in memory. |
| down\_chunks |  | Total number of Chunks that are stored in the filesystem but not loaded in memory yet. |
| busy\_chunks |  | Chunks marked as busy \(being flushed\) or locked. Busy Chunks are immutable and likely are ready to \(or being\) processed. |
|  | size | Amount of bytes used by the Chunk. |
|  | size err | Number of Chunks in an error state where it size could not be retrieved. |

## Storage Layer Dump

Fluent Bit relies on a custom storage layer interface designed for hybrid buffering. The `Storage Layer` entry contains a total summary of Chunks registered by Fluent Bit:

| Entry | Sub-Entry | Description |
| :--- | :--- | :--- |
| total chunks |  | Total number of Chunks |
| mem chunks |  | Total number of Chunks memory-based |
| fs chunks |  | Total number of Chunks filesystem based |
|  | up | Total number of filesystem chunks up in memory |
|  | down | Total number of filesystem chunks down \(not loaded in memory\) |

