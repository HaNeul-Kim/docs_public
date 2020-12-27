# Launchctl

* MacOS 의 systemctl

## plist 작성

```shell script
vim ~/Library/LaunchAgents/com.tistory.hskimsky.ChatCollector.plist
```

### Example

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>com.tistory.hskimsky.ChatCollector</string>
    <key>ProgramArguments</key>
    <array>
      <string>/usr/local/tomcat/songbom/bin/startup.sh</string>
    </array>
    <key>KeepAlive</key>
    <true/>
    <key>RunAtLoad</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/Users/cloudine/test/launchctl/chat_collector.out</string>
    <key>StandardErrorPath</key>
    <string>/Users/cloudine/test/launchctl/chat_collector.err</string>
  </dict>
</plist>
```

```shell script
launchctl load ~/Library/LaunchAgents/com.tistory.hskimsky.ChatCollector.plist
launchctl unload ~/Library/LaunchAgents/com.tistory.hskimsky.ChatCollector.plist
launchctl start com.tistory.hskimsky.ChatCollector
launchctl stop com.tistory.hskimsky.ChatCollector
launchctl list
```

### [manpage](https://www.manpagez.com/man/5/launchd.plist/)

#### Label <string>
#### Disabled <boolean>
#### UserName <string>
#### GroupName <string>
#### inetdCompatibility <dictionary>
##### Wait <boolean>
#### LimitLoadToHosts <array of strings>
#### LimitLoadFromHosts <array of strings>
#### LimitLoadToSessionType <string>
#### Program <string>
#### ProgramArguments <array of strings>
#### EnableGlobbing <boolean>
#### EnableTransactions <boolean>
#### OnDemand <boolean>
#### KeepAlive <boolean or dictionary of stuff>

This optional key is used to control whether your job is to be kept continuously running or to let demand and conditions control the invocation. The default is false and therefore only demand will start the job. The value may be set to true to unconditionally keep the job alive. Alternatively, a dictionary of conditions may be specified to selectively control whether launchd keeps a job alive or not. If multiple keys are provided, launchd ORs them, thus providing maximum flexibility to the job to refine the logic and stall if necessary. If launchd finds no reason to restart the job, it falls back on demand based invocation.  Jobs that exit quickly and frequently when configured to be kept alive will be throttled to converve system resources.

##### SuccessfulExit <boolean>

If true, the job will be restarted as long as the program exits and with an exit status of zero.  If false, the job will be restarted in the inverse condition.  This key implies that "RunAtLoad" is set to true, since the job needs to run at least once before we can get an exit status.

##### NetworkState <boolean>
##### PathState <dictionary of booleans>
##### OtherJobEnabled <dictionary of booleans>
#### RunAtLoad <boolean>
#### RootDirectory <string>
#### WorkingDirectory <string>
#### EnvironmentVariables <dictionary of strings>
#### Umask <integer>
#### TimeOut <integer>
#### ExitTimeOut <integer>
#### ThrottleInterval <integer>
#### InitGroups <boolean>
#### WatchPaths <array of strings>
#### QueueDirectories <array of strings>
#### StartOnMount <boolean>
#### StartInterval <integer>
#### StartCalendarInterval <dictionary of integers or array of dictionary of integers>
##### Minute <integer>
##### Hour <integer>
##### Day <integer>
##### Weekday <integer>
##### Month <integer>
#### StandardInPath <string>
#### StandardOutPath <string>
#### StandardErrorPath <string>
#### Debug <boolean>
#### WaitForDebugger <boolean>
#### SoftResourceLimits <dictionary of integers>
#### HardResourceLimits <dictionary of integers>
##### Core <integer>
##### CPU <integer>
##### Data <integer>
##### FileSize <integer>
##### MemoryLock <integer>
##### NumberOfFiles <integer>
##### NumberOfProcesses <integer>
##### ResidentSetSize <integer>
##### Stack <integer>
#### Nice <integer>
#### ProcessType <string>
Background|Standard|Adaptive|Interactive
#### AbandonProcessGroup <boolean>
#### LowPriorityIO <boolean>
#### LaunchOnlyOnce <boolean>
#### MachServices <dictionary of booleans or a dictionary of dictionaries>
##### ResetAtClose <boolean>
##### HideUntilCheckIn <boolean>
#### Sockets <dictionary of dictionaries... OR dictionary of array of dictionaries...>
##### SockType <string>
##### SockPassive <boolean>
##### SockNodeName <string>
##### SockServiceName <string>
##### SockFamily <string>
##### SockProtocol <string>
##### SockPathName <string>
##### SecureSocketWithKey <string>
##### SockPathMode <integer>
##### Bonjour <boolean or string or array of strings>
##### MulticastGroup <string>