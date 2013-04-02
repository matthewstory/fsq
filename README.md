===
FSQ
===

fsq(7) is a standard for implementing queueing structures in POSIX file-systems.  fsq provides a standard for both directory layouts and work-item naming, which allow for idempotent work-item construction, atomic enqueueing, and atomic completion.

fsq(1) is an interface for the file-system based queue-processing standard fsq(7).  fsq(1) provides a variety of commands for enqueueing work, dispatching work and introspecting, auditing and repairing enqueued, failed, and finished work.


Quick Overview
==============

Each fsq queue conforms to the following directory structure:

    **/var/fsq/a_queue/**

    The  queue a_queue is installed to the default FSQ_ROOT, /var/fsq/.  To enqueue a file to this queue, use the enqueue function:

    ```fs.enqueue('a_queue', ´/path/to/file', 'args', 'to', 'enqueue')```

  or to enqueue a string, use the senqueue function:

    ```fsq.senqueue('a_queue', ´a string body for my work-item', 'args', 'to', 'enqueue')```

    **/var/fsq/a_queue/queue**

    The queue directory within a_queue is the location where work-items are queued to.  Following  the  above enqueue or senqueue function calls, you should be able to see 2 files in the q_queue/queue directory:

``` 
_20120710213904_0_13044_mss_0_args_to_enqueue
_20120710213904_1_13044_mss_0_args_to_enqueue
+|-----+------| + |-+-| |+| + |------+------|
|      |        |   |    |  |        |
|      |        |   |    |  |        +-> FSQ_DELIMITER seperated
|      |        |   |    |  |            arguments
|      |        |   |    |  +-> tries: number of failed attempts
|      |        |   |    |      to process
|      |        |   |    +-> hostname: the name of the host on
|      |        |   |        which the work-item was enqueued.
|      |        |   +-> pid of the process which enqueued the
|      |        |       work-item
|      |        +-> entropy: should a work-item be generated
|      |            with the same arguments, pid, hostname
|      |            and timestamp, entropy is incremented to
|      |            generate uniqueness.
|      +-> timestamp in FSQ_TIMEFMT format
+-> FSQ_DELIMITER used at enqueue time
```

**/var/fsq/a_queue/tmp**
  The tmp directory within a_queue is a location for constructing work-items prior to enqueueing them to the queue directory. In the above enqueue and senqueue calls, the work-item files were initially con‐structed in tmp, then linked into queue and the tmp entry was removed.

**/var/fsq/a_queue/done**
  The  done  directory within a_queue is a location for storing successfully completed work-items following work.  Items may be marked as done, by using the done function (typically done during a scan):

    ``for i in fsq.scan('a_queue'): fsq.done(i)``

  The function success may also be used:

    ``for i in fsq.scan('a_queue'): fsq.success(i)``

  The done directory serves as a sort of book-keeping, but it should routinely be pruned, via cron or some such.


/var/fsq/a_queue/fail
  The  fail  directory within a_queue is a location for storing failed work-items following work. Items may be marked as failed, by using the fail function:

``
    for i in fsq.scan('a_queue'):
      fsq.fail(i)

  The fail_perm function may also be used:

    for i in fsq.scan('a_queue'):
      try:
        raise Exception
      except:
        fsq.fail_perm(i)
``

  permanent failure may also result if an work-item is older than, FSQ_TTL or if the work-item has been retried more than FSQ_MAX_TRIES by way of fail_tmp:

``
    for i in fsq.scan('a_queue'):
      try:
        raise Exception
      except:
        fsq.fail_tmp(i)

  Alternatively, retry may be used:

    for i in fsq.scan('a_queue'):
      try:
        raise Exception
      except:
        fsq.retry(i)
``


