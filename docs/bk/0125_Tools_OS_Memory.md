-----

| Title     | Tools OS Memory                                      |
| --------- | ---------------------------------------------------- |
| Created @ | `2023-10-26T02:45:07Z`                               |
| Updated @ | `2023-10-26T02:45:07Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/125) |

-----

# Memory 监测工具

  - top
  - htop
  - free

## 查看应用最大使用内存

    /usr/bin/time --verbose [your_app]

> example: `/usr/bin/time --verbose du -h`

``` 

        Command being timed: "xxxxx"
        User time (seconds): 33.65
        System time (seconds): 2.73
        Percent of CPU this job got: 738%
        Elapsed (wall clock) time (h:mm:ss or m:ss): 0:04.92
        Average shared text size (kbytes): 0
        Average unshared data size (kbytes): 0
        Average stack size (kbytes): 0
        Average total size (kbytes): 0
      * Maximum resident set size (kbytes): 3190636
        Average resident set size (kbytes): 0
        Major (requiring I/O) page faults: 78
        Minor (reclaiming a frame) page faults: 726357
        Voluntary context switches: 472
        Involuntary context switches: 222
        Swaps: 0
        File system inputs: 2760
        File system outputs: 0
        Socket messages sent: 0
        Socket messages received: 0
        Signals delivered: 0
        Page size (bytes): 4096
        Exit status: 0
```

  - **Maximum resident set size** : 实际使用物理内存（包含共享库占用的全部内存）
