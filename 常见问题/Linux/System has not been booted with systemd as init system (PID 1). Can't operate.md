# [System has not been booted with systemd as init system (PID 1). Can't operate](https://askubuntu.com/questions/1379425/system-has-not-been-booted-with-systemd-as-init-system-pid-1-cant-operate)

[Ask Question](https://askubuntu.com/questions/ask)

Asked 1 year, 1 month ago

Modified [1 month ago](https://askubuntu.com/questions/1379425/system-has-not-been-booted-with-systemd-as-init-system-pid-1-cant-operate?lastactivity "2022-12-05 22:16:14Z")

Viewed 385k times

106

[](https://askubuntu.com/posts/1379425/timeline)

I use WSL2 on Windows 11. I want to run the `systemctl` command in Ubuntu 20.04, but it gives me the following error:

```
System has not been booted with systemd as init system (PID 1). 
Can't operate. Failed to connect to bus: Host is down
```

How can I fix it?

-   [boot](https://askubuntu.com/questions/tagged/boot "show questions tagged 'boot'")
-   [windows-10](https://askubuntu.com/questions/tagged/windows-10 "show questions tagged 'windows-10'")
-   [windows-subsystem-for-linux](https://askubuntu.com/questions/tagged/windows-subsystem-for-linux)

[Share](https://askubuntu.com/q/1379425 "Short permalink to this question")

[Improve this question](https://askubuntu.com/posts/1379425/edit)

Follow

[edited Jul 26, 2022 at 13:05](https://askubuntu.com/posts/1379425/revisions "show all edits to this post")

[

![NotTheDr01ds's user avatar](media/NotTheDr01ds's_user_avatar.jpg)

](https://askubuntu.com/users/1165986/notthedr01ds)

[NotTheDr01ds](https://askubuntu.com/users/1165986/notthedr01ds)

11.6k22 gold badges4242 silver badges6868 bronze badges

asked Dec 6, 2021 at 0:37

[

![dsha256's user avatar](media/dsha256's_user_avatar.jpg)

](https://askubuntu.com/users/1549536/dsha256)

[dsha256](https://askubuntu.com/users/1549536/dsha256)

1,18822 gold badges44 silver badges1010 bronze badges

-   5
    
    The `systemctl` command won't work on WSL without some serious hacking. Not recommended. 
    
    – [user535733](https://askubuntu.com/users/19626/user535733 "51,668 reputation")
    
     [Dec 6, 2021 at 0:49](https://askubuntu.com/questions/1379425/system-has-not-been-booted-with-systemd-as-init-system-pid-1-cant-operate#comment2377631_1379425) 
    
-   Why would you want to use `systemctl` on Windows 11? You should be using Windows commands. What are you trying to achieve? 
    
    – [WinEunuuchs2Unix](https://askubuntu.com/users/307523/wineunuuchs2unix "97,225 reputation")
    
     [Dec 6, 2021 at 1:04](https://askubuntu.com/questions/1379425/system-has-not-been-booted-with-systemd-as-init-system-pid-1-cant-operate#comment2377633_1379425)
    
-   3
    
    @WinEunuuchs2Unix The OP is running Ubuntu in WSL2. The question is not about running systemctl in Windows but in Ubuntu. That said user535733 is absolutely right. 
    
    – [ChanganAuto](https://askubuntu.com/users/1210606/changanauto "1,941 reputation")
    
     [Dec 6, 2021 at 1:06](https://askubuntu.com/questions/1379425/system-has-not-been-booted-with-systemd-as-init-system-pid-1-cant-operate#comment2377635_1379425)
    
-   2
    
    Try using the `service` command instead. 
    
    – [user535733](https://askubuntu.com/users/19626/user535733 "51,668 reputation")
    
     [Dec 6, 2021 at 1:26](https://askubuntu.com/questions/1379425/system-has-not-been-booted-with-systemd-as-init-system-pid-1-cant-operate#comment2377642_1379425)
    
-   3
    
    Specifically, `sudo service ssh restart`. But be aware that `ssh` doesn't quite work as you might expect under WSL2 either. 
    
    – [NotTheDr01ds](https://askubuntu.com/users/1165986/notthedr01ds "11,550 reputation")
    
     [Dec 6, 2021 at 1:52](https://askubuntu.com/questions/1379425/system-has-not-been-booted-with-systemd-as-init-system-pid-1-cant-operate#comment2377659_1379425)
    

[Show **4** more comments](https://askubuntu.com/questions/1379425/system-has-not-been-booted-with-systemd-as-init-system-pid-1-cant-operate# "Expand to show all comments on this post")

## 2 Answers

Sorted by:

                                              Highest score (default)                                                                   Date modified (newest first)                                                                   Date created (oldest first)                              

128

[](https://askubuntu.com/posts/1379567/timeline)

Surprisingly, after 6 years or so of WSL, there doesn't seem to be a good, general-purpose "Systemd" question here on Ask Ubuntu. So this looks like a good one to use for that purpose.

In general, when you see _either_ of the following two messages:

-   `System has not been booted with systemd as init system (PID 1). Can't operate.`
-   `Failed to connect to bus: Host is down`

Then it's typically the same root cause. In the case of `systemctl` and attempting to start `ssh`, you are seeing both.

The problem may be that:

-   Your release of WSL doesn't support Systemd. In this case, there are multiple workarounds available. See the _Alternatives to Systemd in WSL_ section below.
    
-   The good news is that Systemd is now officially supported in Ubuntu on many WSL systems. See below for how to determine if your system supports it and how to enable it (if you need it).
    

#### Should you enable Systemd in WSL?

First, consider whether you _should_ enable Systemd in WSL. Enabling Systemd will automatically start a lot of background services and tasks that you really may not need under WSL. As a result, it will also increase WSL startup times, although the impact will be dependent on your system. Check the _Alternatives_ section below to see if there may be a better option that fits your needs. For example, the `service` command may do what you need without any additional effort.

While I'm happy that Systemd is available as an option, I personally plan on continuing to run without it whenever possible.

#### How to enable Systemd in Ubuntu/WSL

As background, there are now two different "delivery mechanisms" (I'll think of a better term, hopefully) for WSL2. I'd call them different "versions" or "releases", but we tend to already use that term for WSL version 1 and 2.

-   Originally, WSL1 and WSL2 both came as a Windows _feature_, which was enabled through the _Turn Windows features on or off_ settings. This feature was (and still is) built-in to Windows, and is called the "in-box" version of WSL.
    
-   In October of 2021, Microsoft started making WSL2 available as a Windows _application_, which could be installed through the Microsoft Store (and other methods described below).
    

It's the WSL _application_ that supports Systemd. Currently, the in-box version of WSL does not support Systemd. To use the new WSL application, you must be on a supported Windows release:

-   New WSL users with Windows 11 22H2 or later will automatically receive the application version of WSL when running `wsl --install`, unless specifically adding the `--inbox` option.
    
-   Windows 11 21H2 users can still install the WSL application using the methods below.
    
-   Windows 10 users will need [KB5020030](https://support.microsoft.com/en-gb/topic/november-15-2022-kb5020030-os-builds-19042-2311-19043-2311-19044-2311-and-19045-2311-preview-237a9048-f853-4e29-a3a2-62efdbea95e2) or later. Note that it is not yet clear at the time of this update whether older Windows 10 releases will work. I have personally only been able to validate it on Windows 10 22H2 so far.
    

With the prerequisite Windows version in place, you can then install or upgrade to the 1.0.0 release (or later) of the WSL application using several methods:

-   Through the Microsoft Store (as "Windows Subsystem for Linux").
    
-   Or from the [Releases](https://github.com/microsoft/WSL/releases) page in the Github repo. To install a release manually:
    
    1.  Reboot (to make sure that WSL is not in use at all). A simple `wsl --shutdown` _may_ work, but often will not.
        
    2.  Download the 1.0.0 (or later) release from the link above.
        
    3.  Start an Administrator PowerShell and:
        
        ```
        Add-AppxPackage <path.to>/Microsoft.WSL_1.0.0.0_x64_ARM64.msixbundle
        wsl --version # to confirm
        ```
        

To enable, start your Ubuntu (or other Systemd) distribution under WSL (typically just `wsl ~` will work).

```
sudo -e /etc/wsl.conf
```

Add the following:

```
[boot]
systemd=true
```

Exit Ubuntu and again:

```
wsl --shutdown
```

Then restart Ubuntu.

```
sudo systemctl status
```

... should show your Systemd services.

---

### Alternatives to Systemd in WSL

`systemctl` is most often used to start services under Ubuntu. For older releases that don't support Systemd (or if you just choose not to enable it), there are still several alternatives that might work in place of the `systemctl` command.

Fortunately, Ubuntu is pretty good overall about being able to cope without Systemd.

#### How to handle the lack of Systemd

Systemd at its core is just a (probably gross-oversimplification) "way of accomplishing system tasks". There is usually (but not always, see footnote below) a way of doing the same task without Systemd, and often more than one way.

-   _**Option 1: "The old way"**_
    
    In Ubuntu on WSL, many of the common system services still have the "old" `init.d` scripts available to be used in place of `systemctl` with Systemd units. You can see these by using `ls /etc/init.d/`.
    
    So, for example, you can start `ssh` with `sudo service ssh start`, and it will run the `/etc/init.d/ssh` script with the `start` argument.
    
    Even some non-default packages such as MySql/MariaDB will install both the Systemd unit files _and_ the old `init.d` scripts, so you can still use the `service` command for them as well.
    
-   _**Option 2: Docker**_
    
    Many packages/services are available as Docker images. Docker runs great under Ubuntu on WSL2 (specifically WSL2; it will not run on WSL1). If there's not a SysVinit "service" script for the service you are trying to start, there may very well be a Docker image available that runs in a containerized environment.
    
    Example: Elasticsearch, as in [this question and my answer](https://askubuntu.com/q/1427872/1165986).
    
    -   Bonus #1: Doesn't interfere with other packages already installed (no dependency issues).
    -   Bonus #2: The Docker images themselves pretty much never use Systemd, so you can often inspect the `Dockerfile` to see how the service is started without Systemd. For more information see the next option - "The manual way."
-   _**Option 3: "The 'manual' way"**_
    
    _Edit: Bumped down notch from its former position as "Option 2", since Docker is probably a better alternative for many services._
    
    But some services don't have a init-script equivalent, especially on other distributions. For simplicity, let's assume that the `ssh` `init.d` script _wasn't_ available.
    
    In this case, the "answer" is to figure out what the Systemd unit files are doing and attempt to replicate that manually. This can vary widely in complexity. But I'd start with looking at the Systemd unit file that you are trying to run:
    
    ```
    less /lib/systemd/system/ssh.service
    ```
    
    ```
    # Trimmed
    [Service]
    EnvironmentFile=-/etc/default/ssh
    ExecStartPre=/usr/sbin/sshd -t
    ExecStart=/usr/sbin/sshd -D $SSHD_OPTS
    RuntimeDirectory=sshd
    RuntimeDirectoryMode=0755
    ```
    
    I've trimmed out some of the less relevant lines for understanding its behavior, but you can `man systemd.exec`, `man systemd.service`, and others to see what most of the options do.
    
    In this case, when you `sudo systemctl start ssh`, it:
    
    -   Reads environment variables (the `$SSHD_OPTS`) from `/etc/default/ssh`
    -   Tests the config, exits if there is a failure
    -   Makes sure the RuntimeDirectory exists with the specified permissions. This translates to `/run/sshd` (from `man systemd.exec`). This also removes the runtime directory when you stop the service.
    -   Runs `/usr/sbin/sshd` with options
    
    So, if you don't have any environment-based config, you could just set up a script to:
    
    -   Make sure the runtime directory exists. Note that, since it is in `/run`, which is a `tmpfs` mount, it will be deleted after every restart of the WSL instance.
    -   Set the permissions to `0755`
    -   Start `/usr/sbin/sshd` as root
    
    ... And you would have done the same thing _manually_ without Systemd.
    
    Again, this is probably the simplest example. You might have much more to work through for more complex tasks.
    
-   _**Option 4: Run Systemd as PID 1 in a PID namespace/container**_
    
    Finally, it's _possible_ to get Systemd running under WSL2 (but not WSL1). This is a fairly advanced topic, although there are multiple scripts and projects that attempt to simplify it.
    
    _**Warnings:** Systemd fundamentally changes many aspects of Ubuntu when started, including the way that X sockets, login, WSL Interop, temp files, and more! Due to the shared VM nature of WSL2, some of these changes can even impact other distributions you are using _without_ Systemd._
    
    _My personal recommendation is to either (a) make sure that you understand what is going on behind the scenes should you use one of these techniques, (b) don't do it!, or (c) at the very least, when asking questions about why something doesn't work, make sure to let people know that you are using a Systemd helper script under WSL (and which one)._
    
    With that out of the way ...
    
    Let's start with some of the more popular projects to enable Systemd in WSL:
    
    -   [Genie](https://github.com/arkane-systems/genie)
    -   [WSL2-Hacks](https://github.com/shayne/wsl2-hacks)
    -   [Distrod](https://github.com/nullpo-head/wsl-distrod)
    
    I don't personally run any of them on a regular basis, but all are open-source, and I've scanned the source to compare the techniques. At the core, each creates a new namespace or container where Systemd can run as PID 1.
    
    You can see this in action by following the steps:
    
    1.  Run:
        
        ```
        sudo -b unshare --pid --fork --mount-proc /lib/systemd/systemd --system-unit=basic.target
        ```
        
        This starts Systemd in a new namespace with its own PID mapping. Inside that namespace, Systemd will be PID1 (as it must, to function) and own all other processes. However, the "real" PID mapping still exists outside that namespace.
        
        Note that this is a "bare minimum" command-line for starting Systemd. It will not have support for, at least:
        
        -   Windows Interop (the ability to run Windows `.exe`)
        -   The Windows PATH (which isn't necessary without Windows Interop anyway)
        
        The scripts and projects listed above do extra work to get these things working as well.
        
    2.  Wait a few seconds for Systemd to start up, then:
        
        ```
        sudo -E nsenter --all -t $(pgrep -xo systemd) runuser -P -l $USER -c "exec $SHELL"
        ```
        
        This enters the namespace, and you can now use `ps -efH` to see that `systemd` is running as PID 1 in that namespace.
        
        At this point, you should be able to run `systemctl`.
        
    3.  And after proving to yourself that it's possible, I recommend exiting all WSL instances completely, then doing `wsl --shutdown`. Otherwise, you will have some things be "broken" until you do. They can likely be "fixed", but that's beyond the scope of any one Ask Ubuntu question ;-). My recommendation is to refer to the projects above to see how they handle it.
        

##### Footnote 1 - Snap

There are certain applications and function in Ubuntu that are just so complex that they are unlikely to ever be disentangled from Systemd. The most obvious example here would be the Snap system. I'm sure it's _theoretically possible_ that a version of the Snap system could be created without the use of Systemd. However, there are two good reasons why this won't happen (anytime soon, at least):

-   Snap is a system created and supported by Canonical.
    
-   Canonical _has_ selected Systemd as the Ubuntu init system. Just because WSL doesn't support it (easily) doesn't change this. Canonical developers wrote the Snap system with the expectation that the functionality of Systemd is present.
    
-   There currently doesn't seem to be any third-party desire to port Snap to non-Systemd distributions. Those distributions typically utilize Flatpak in place of Snap. It should be noted, however, that even Flatpak tends to utilize Systemd on distributions where it is available. However, Flatpak is also available for non-Systemd distros.
    

##### Footnote 2 - Gnome

Gnome is also an application (ecosystem) that is very tightly coupled with Systemd, but it's popular enough that there _have_ been ports to non-Systemd init systems. That said, running it on Ubuntu does assume the presence of Systemd, so you would have to reverse-engineer the process used on those other distributions if you wanted to run it on Ubuntu/WSL without the use of Systemd.

##### Footnote 3 - Other Systemd-dependent applications

There are also cases where certain software _expects_ Systemd and just won't work (or at least not fully) if it isn't present. One that I came across recently is [Cockpit](https://cockpit-project.org/).

While I was able to get it up and running without Systemd, it ultimately expects Systemd to be present in order to execute many, if not most, functions. In effect, parts of Cockpit are a front-end to Systemd.

This type of software, one that _executes_ Systemd commands (such as `systemctl`), may be the exception to the "there's an alternative" rule.