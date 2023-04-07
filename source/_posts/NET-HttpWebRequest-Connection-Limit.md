---
title: .NET HttpWebRequest Connection Limit
date: 2023-04-07 22:00:00
categories:
- [dev, csharp]
tags:
- csharp
- dot_net
---

## Trap

Recently I stuck in a trap. There is a performance issue in the project I'm taking responsibility for. There is an interface in the project. As a .NET client, it would send quite a few keys to server and would receive the resources corresponding to the keys one by one.

Some customers gave us some feedback that the resources fetching costed very long time and it affected customers' productivity unacceptably. Unfortunately, we're not able to gain the actual quantity of keys in customers' usage environment, but fortunately we found a way to reproduce the issue and recorded it by Windows Performance Recorder(WPR) though it costed at least 10 minutes.

The keys have been separated into multiple groups by every 200 keys as a group. Every group would be sent to server parallelly. From this perspective, the performance issue are beyond our expectations.

## Profiling Call Stack

After profiling in Windows Performance Analyzer(WPA), the call stack within threads showed some clues. Many threads' wait time are pretty long. It means that they are blocked by IO or lock. With further inquiry, in most cases they are blocked inside `SendRequest` but before the actual request sending. We have to throw a question - what block these threads and make them waiting? Luckily, WPA shows the call stack of the ready thread(s) next to every blocked thread. Every ready thread is recorded at the socket `read` stage, and they are also blocked at `SendRequest` by other threads in `read` shown in the WPA call stack.

Based on it, I conjectured that the thread leverages a lock and releases the lock in `read`. After that, the waiting thread would gets the lock and then does sending its request. I have to mention that those threads follow one by one and they construct a dependent chain.

I suspected there is a thread pool inside the .NET network framework, after all there is a function named `WaitOne` about thread. However, I couldn't find any keyword about thread pool by reviewing the code context through the call stack.

In the end, I noticed the type of request is `HttpWebRequest` and tried to search it in the Internet. Fortunately, there is another village in the dark.

## HttpWebRequest Connection limitation

**For `HttpWebRequest`, it has a concurrent connection limit that is `2` by default.** It approximates no concurrency! No wonder the pool threads were blocked one by one...

The solution to fix it isn't hard. We just need to increase the connection limit to a reasonable value such as `512`.Actually we have 2 ways to achieve it:

### Adjust in Configuration

1. Open the configuration file `App.config` in the project.
2. The original configuration might like this:

    ```XML
    <?xml version="1.0" encoding="utf-8" ?>
    <configuration>
        <startup> 
            <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.6" />
        </startup>
    </configuration>
    ```

3. Add a field scope named `connectionManagement` and add a value about `maxconnection`.

    ```XML
    <?xml version="1.0" encoding="utf-8" ?>
    <configuration>
        <startup> 
            <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.6" />
        </startup>
        <system.net>
            <defaultProxy enabled="false">
                <proxy/>
                <bypasslist/>
                <module/>
            </defaultProxy>
            <connectionManagement>
                <add address="*" maxconnection="512"/>
            </connectionManagement>
        </system.net>
    </configuration>
    ```

### Adjust in Program

Add a line in code like below:

```C#
System.Net.ServicePointManager.DefaultConnectionLimit = 512;
```

## Other Informations

As for `HttpClient` after .NET 4.5, it seems that it doesn't have the connection limit. **Note that** it's different to `HttpWebRequest`.

## References

- [Microsoft .NET Framework 4.8 Documentation](https://learn.microsoft.com/en-us/dotnet/api/system.net.servicepointmanager.defaultconnectionlimit?view=netframework-4.8#System_Net_ServicePointManager_DefaultConnectionLimit)
- <https://blog.csdn.net/PLA12147111/article/details/105496791>
- <https://blog.csdn.net/defender_/article/details/91949613>
