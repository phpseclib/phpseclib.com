---
id: commands
title: Running Commands
---

## One-off commands

```php
echo $ssh->exec('pwd');
echo $ssh->exec('ls -la');
```

By default `$ssh->exec()` returns both stdout and stderr. To suppress stderr you can call `$ssh->enableQuietMode()`. To re-enable it call `$ssh->disableQuietMode()`.

To get stderr separately from stdout you'll need to call `$ssh->enableQuietMode()` and then call `$ssh->getStdError()`. These functions do _not_ work with `$ssh->read()` (ie. when a PTY is enabled) for reasons described [here](https://superuser.com/a/1581769/172193).

To get the exit status you can call `$ssh->getExitStatus()`.

To see whether or not "quiet mode" is enabled do `$ssh->isQuietModeEnabled()`.

### Callbacks

```php
$ssh->exec('ping 127.0.0.1', function($str) {
    echo $str;
    //if (strpos($str, 'icmp_seq=5') !== false) {
    //    return true;
    //}
});
```

This example is best run via the CLI. Run it via the webbrowser and you may need to [flush](http://php.net/flush) the output buffer and even then YMMV.

The commented out code shows how you could use a callback method to return early based on whatever your criteria might be. ie. if the callback method returns `bool(true)` then `exec()` will return early.

### Gotcha: Successive calls to exec()

```php
echo $ssh->exec('pwd'); // outputs /home/username
$ssh->exec('cd /');
echo $ssh->exec('pwd'); // (despite the previous command) outputs /home/username
```

If done on an interactive shell, the output you'd receive for the first pwd would (depending on how your system is setup) be different than the output of the second pwd. The above code snippet, however, will yield two identical lines.

The reason for this is that any "state changes" you make to the one-time shell are gone once the exec() has been ran and the channel has been deleted.

You can workaround this on Linux by doing `$ssh->exec('cd /; pwd')` or `$ssh->exec('cd / && pwd')`.

## Interactive Shell

```php
echo $ssh->read('username@username:~$');
$ssh->write("ls -la\n"); // note the "\n"
echo $ssh->read('username@username:~$');
```

On a terminal you normally don't just type the command and expect to get the output. You type the command and then hit enter. To simulate that you'll need to add `"\n"` to the commands you send via `write()`

### Gotcha: Writing without first reading

You should always `$ssh->read()` the prompt _before_ `$ssh->write()`'ing the command. Consider the following:

```php
$ssh = new SSH2('localhost');
$ssh->login('user', 'pass');

$ssh->write("ping -c 5 127.0.0.1\n");
echo $ssh->read('username@username:~$');
```
In this example the initial `$ssh->read('username@username:~$')` is being skipped. Superficially, this might seem reasonable enough, but in practice, it won't work.

Here's the output of the code, as written:

```
ping -c 5 127.0.0.1
Last login: Thu Feb  6 06:51:26 2020 from 10.0.2.2
username@username:~$
```
Here's the output if `$ssh->read('username@username:~$');` had been done before:

```
ping -c 5 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.015 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.038 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.074 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.038 ms
64 bytes from 127.0.0.1: icmp_seq=5 ttl=64 time=0.044 ms

--- 127.0.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4093ms
rtt min/avg/max/mdev = 0.015/0.041/0.074/0.020 ms
username@username:~$
```

### Gotcha: echo'ing out stdin

Let's say you wanted to "stream" a bunch of bytes into a file via stdin. Superficially one might think this would work:

```
$ssh->exec('cat > filename.ext');
$ssh->write("asdfasdf\n");
```
For non-binary data it does but it does so with a few caveats.

1. Any data you send to it via `$ssh->write()` will be echo'd out, so you'll also want to do `$ssh->read('asdfasdf')`. This makes sense when you consider that there are times when Linux does _not_ echo out the keys you type. eg. when you're in [normal mode](https://en.wikibooks.org/wiki/Learning_the_vi_Editor/Vim/Modes#normal_(command)) in vim. If you hit <kbd>i</kbd> or <kbd>a</kbd> you'll enter into  [insert mode](https://en.wikibooks.org/wiki/Learning_the_vi_Editor/Vim/Modes#insert_(and_replace)) without those characters being echo'd out. Whether or not the keystrokes you enter are echo'd out to you depends on the context.
2. This doesn't work well with binary data. The [PuTTYPuTTYPuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/faq.html#faq-puttyputty) question in the PuTTY FAQ addresses this.

## read() with regular expressions: sudo

Technically, [sudo](http://en.wikipedia.org/wiki/Sudo) is as easy as doing `echo 'password' | sudo -S command` but sudo also provides a good example of how to use regular expressions with `$ssh->read()`:

```php
echo $ssh->read('username@username:~$');
$ssh->write("sudo ls -la\n");
$output = $ssh->read('#[pP]assword[^:]*:|username@username:~\$#', SSH2::READ_REGEX);
echo $output;
if (preg_match('#[pP]assword[^:]*:#', $output)) {
    $ssh->write("password\n");
    echo $ssh->read('username@username:~$');
}
```

By default, sudo caches passwords for 5 minutes after they've been entered. So while `$ssh->read('Password:')` will work the first time you try it, it won't work if you try it within a five minutes after having initially ran it.

## setTimeout()

By default, phpseclib should timeout if the command you're running takes more than 10s. You can change this behavior thusly:

```php
$ssh->setTimeout(1);
echo $ssh->read();
```

After each `$ssh->read()` the timeout resets to what you set it to last. By default the "channel" remains open so if you do `$ssh->write("ping 127.0.0.1\n")` and do two successive `$ssh->read()` you'll get ping output in both of those calls. If you don't want it to do that you can either send Ctrl + C via `$ssh->write("\x03")` or you can reset the whole channel by calling `$ssh->reset()`.

`$ssh->isTimeout()` will return true if the result of the last `$ssh->read()` or `$ssh->exec()` was due to a timeout. Otherwise it will return false.

`$ssh->setTimeout()` works with `$ssh->read()` and `$ssh->exec()`. If an `$ssh->exec()` call times out and you want to run another `$ssh->exec()` call after the timed out one, do `$ssh->reset()`.

## setKeepAlive()

In some cases it may be necessary to send a "[keepalive](https://en.wikipedia.org/wiki/Keepalive)" to the server every x seconds to prevent the SSH connection from being closed by the server during long running commands. eg. if, in the target servers sshd_config, ClientAliveCountMax is 0 and ClientAliveInterval is 3, then `echo $ssh->exec('sleep 10; ls -latr')` will timeout and you'll get a "Connection closed prematurely" ConnectionClosedException. `setKeepAlive(...)` is the answer to this problem.

Note that `setKeepAlive()` will not keep a connection alive if you're doing a time consuming operation _outside_ of the SSH instance. eg. doing `sleep(10); echo $ssh->exec('ls -latr');` on a server with the aforementioned sshd_config will still result in a "Connection closed prematurely" ConnectionClosedException because, at the end of the day, PHP is still a synchronous language and unless phpseclib has control then the "keepalive" packets won't be sent out.

## Determining what to read(): passwd

Let's say you want to change your password. Technically, SSH has [built-in OS-independent functionality](https://tools.ietf.org/html/rfc4252#page-11) to accomodate this but phpseclib doesn't (currently) support it so we'll just use [passwd](https://en.wikipedia.org/wiki/Passwd). So how would that work? Normally you're presented with a series of prompts but what are those prompts? Let's find out.

```php
$ssh->enablePTY();
$ssh->setTimeout(3);
$ssh->exec('passwd');
echo $ssh->read();
```
This outputs `(current) UNIX password` so we'll wait for `password:` to appear. But what comes after that?

```php
$ssh->enablePTY();
$ssh->setTimeout(3);
$ssh->exec('passwd');
$ssh->setTimeout(3);
$ssh->read('password:');
$ssh->write("oldpassword\n");
echo $ssh->read();
```
This outputs `Enter new UNIX password:` so we'll, once again, wait for `password:` to appear. But what about after that?

```php
$ssh->enablePTY();
$ssh->setTimeout(3);
$ssh->exec('passwd');
$ssh->setTimeout(3);
$ssh->read('password:');
$ssh->write("oldpassword\n");
$ssh->read('password:');
$ssh->write("newpassword\n");
echo $ssh->read();
```
This outputs `Retype new UNIX password:` so we'll, once again, wait for `password:` to appear. But what about after that?

```php
$ssh->enablePTY();
$ssh->setTimeout(3);
$ssh->exec('passwd');
$ssh->setTimeout(3);
$ssh->read('password:');
$ssh->write("oldpassword\n");
$ssh->read('password:');
$ssh->write("newpassword\n");
$ssh->read('password:');
$ssh->write("newpassword\n");
echo $ssh->read();
```
This outputs `passwd: password updated successfully` so I guess we're done! At this point `$ssh->setTimeout(3)` is unneccessary. The last `$ssh->read()`, in my testing, returns immediately.

## ANSI Escape Codes

```php
use phpseclib3\File\ANSI;

$ansi = new ANSI;

$ansi->appendString($ssh->read('username@username:~$'));
$ssh->write("top\n");
$ssh->setTimeout(5);
$ansi->appendString($ssh->read());
echo $ansi->getScreen(); // outputs HTML
```

Some commands issued to a terminal may yield [ANSI escape codes](http://en.wikipedia.org/wiki/ANSI_escape_code). eg. `^[[H`. These provide the terminal with information on the formating of the characters and their positioning.

Since \phpseclib3\Net\SSH2 uses vt100 as the "TERM environment variable value" a [VT100](http://en.wikipedia.org/wiki/VT100) [terminal emulator](http://en.wikipedia.org/wiki/Terminal_emulator) is needed to properly handle the ANSI escape codes. \phpseclib3\File\ANSI aims to be such an emulator. The default screen size is 80x24.

`$ansi->getScreen()` returns what'd be seen on the current screen. In the case of top this is desirable as it'll produce output like this:

<pre style="color: white; background: black; font-family: Courier New, Courier, Lucida Sans Typewriter, Lucida Typewriter, monospace; font-size: 13px; line-height: 13px">top - 23:39:24 up 77 days,  1:13,  1 user,  load average: 0.00, 0.00, 0.00
Tasks:  45 total,   2 running,  43 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.0%us,  0.0%sy,  0.0%ni,100.0%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:   1740956k total,  1079288k used,   661668k free,   221240k buffers
Swap:        0k total,        0k used,        0k free,   399940k cached
&nbsp;
<span style="color: black; background: white">  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND            </span>
    1 root      16   0  2128  696  600 S  0.0  0.0   0:01.10 init               
    2 root      RT   0     0    0    0 S  0.0  0.0   0:00.00 migration/0        
    3 root      34  19     0    0    0 S  0.0  0.0   0:00.05 ksoftirqd/0        
    4 root      RT   0     0    0    0 S  0.0  0.0   0:00.01 watchdog/0         
    5 root      10  -5     0    0    0 S  0.0  0.0   0:00.25 events/0           
    6 root      11  -5     0    0    0 S  0.0  0.0   0:00.63 khelper            
    7 root      10  -5     0    0    0 S  0.0  0.0   0:00.00 kthread            
    8 root      14  -5     0    0    0 S  0.0  0.0   0:00.00 xenwatch           
    9 root      10  -5     0    0    0 S  0.0  0.0   0:00.00 xenbus             
   17 root      10  -5     0    0    0 S  0.0  0.0   0:00.00 kblockd/0          
   46 root      20  -5     0    0    0 S  0.0  0.0   0:00.00 aio/0              
   45 root      15   0     0    0    0 S  0.0  0.0   0:00.64 kswapd0            
  562 root      20  -5     0    0    0 S  0.0  0.0   0:00.00 kseriod            
  657 root      15   0     0    0    0 S  0.0  0.0   0:04.23 kjournald          
  718 root      13  -4  2360  656  424 S  0.0  0.0   0:00.18 udevd              
 1592 root      14  -2  2396  848  560 S  0.0  0.0   0:00.03 dhclient           
 1647 root      15   0     0    0    0 S  0.0  0.0   0:00.16 kjournald</pre>

In the case of [ls](http://en.wikipedia.org/wiki/Ls), however, it is less desirable. For commands like ls it may be preferable to do `$ansi->getHistory()`. For top, that'd return the following:

<pre style="color: white; background: black; font-family: Courier New, Courier, Lucida Sans Typewriter, Lucida Typewriter, monospace; font-size: 13px; line-height: 13px">         __|  __|_  )  Fedora 8
         _|  (     /    32-bit
        ___|\___|___|
&nbsp;
 Welcome to an EC2 Public Image
                       :-)
&nbsp;
   Base
&nbsp;
 --[ see /etc/ec2/release-notes ]--
&nbsp;
[username@username:~$]$top
&nbsp;
top - 23:51:56 up 77 days,  1:25,  1 user,  load average: 0.00, 0.00, 0.00
Tasks:  45 total,   2 running,  43 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.0%us,  0.0%sy,  0.0%ni,100.0%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:   1740956k total,  1079256k used,   661700k free,   221240k buffers
Swap:        0k total,        0k used,        0k free,   399940k cached
&nbsp;
<span style="color: black; background: white">  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND            </span>
    1 root      16   0  2128  696  600 S  0.0  0.0   0:01.10 init               
    2 root      RT   0     0    0    0 S  0.0  0.0   0:00.00 migration/0        
    3 root      34  19     0    0    0 S  0.0  0.0   0:00.05 ksoftirqd/0        
    4 root      RT   0     0    0    0 S  0.0  0.0   0:00.01 watchdog/0         
    5 root      10  -5     0    0    0 S  0.0  0.0   0:00.25 events/0           
    6 root      11  -5     0    0    0 S  0.0  0.0   0:00.63 khelper            
    7 root      10  -5     0    0    0 S  0.0  0.0   0:00.00 kthread            
    8 root      14  -5     0    0    0 S  0.0  0.0   0:00.00 xenwatch           
    9 root      10  -5     0    0    0 S  0.0  0.0   0:00.00 xenbus             
   17 root      10  -5     0    0    0 S  0.0  0.0   0:00.00 kblockd/0          
   46 root      20  -5     0    0    0 S  0.0  0.0   0:00.00 aio/0              
   45 root      15   0     0    0    0 S  0.0  0.0   0:00.64 kswapd0            
  562 root      20  -5     0    0    0 S  0.0  0.0   0:00.00 kseriod            
  657 root      15   0     0    0    0 S  0.0  0.0   0:04.23 kjournald          
  718 root      13  -4  2360  656  424 S  0.0  0.0   0:00.18 udevd              
 1592 root      14  -2  2396  848  560 S  0.0  0.0   0:00.03 dhclient           
 1647 root      15   0     0    0    0 S  0.0  0.0   0:00.16 kjournald</pre>

The history, by default, stores 200 lines (not including the current screen). This can be changed by doing `$ansi->setHistory(100)` or whatever.

Both functions return HTML with the various formatting properties specified by HTML. If you don't want HTML and want just the raw text of the terminal without any formatting do `htmlspecialchars_decode(strip_tags($ansi->getScreen()))`.

## Sending Special Characters

```php
use phpseclib3\File\ANSI;

$ssh->setTimeout(2);
$ssh->read();
$ssh->write("vim\n");
$ssh->read();
$ssh->write("\x1BOP"); // send F1

$ansi = new ANSI;
$ansi->appendString($ssh->read());
```

A table of special characters and the keys they correspond to can be found at [SSH2 Special Characters](special-chars.md). The output of the above program is as follows:

<pre style="color: white; background: black; font-family: Courier New, Courier, Lucida Sans Typewriter, Lucida Typewriter, monospace; font-size: 13px; line-height: 13px">*<u>help.txt</u>*      For <u>Vim version 7.3.</u>  Last change: 2010 Jul 20
&nbsp;
  VIM - main help file
  k
      Move around:  Use the cursor keys, or &quot;h&quot; to go left, h   l
  &quot;j&quot; to go down, &quot;k&quot; to go up, &quot;l&quot; to go right.       j
Close this window:  Use &quot;:q<b>&lt;Enter&gt;</b>&quot;.
   Get out of Vim:  Use &quot;:qa!<b>&lt;Enter&gt;</b>&quot; (careful, all changes are lost!).
&nbsp;
Jump to a subject:  Position the cursor on a tag (e.g. |<u>bars</u>|) and hit <b>CTRL-]</b>.
   With the mouse:  &quot;:set mouse=a&quot; to enable the mouse (in xterm or GUI).
  Double-click the left mouse button on a tag, e.g. |<u>bars</u>|.
        Jump back:  Type <b>CTRL-T</b> or <b>CTRL-O</b> (repeat to go further back).
&nbsp;
Get specific help:  It is possible to go directly to whatever you want help
     on, by giving an argument to the |<u>:help</u>| command.
     It is possible to further specify the context:
                                         *<u>help-context</u>*
  <u>WHAT                  PREPEND    EXAMPLE</u>      ~
  Normal mode command      (nothing)   :help x
<span style="color: black; background: white">help.txt [Help][RO]                                           1,1            Top</span>
&nbsp;
<span style="color: black; background: white">[No Name]                                                     0,0-1          All</span>
&quot;help.txt&quot; [readonly] 221L, 8239C</pre>

The output may look different than vim when ran through, for example, PuTTY, because PuTTY uses `xterm` as it's shell whereas phpseclib uses `vt100`.

## exec() with and without a PTY vs Interactive Shells

### passwd

Some commands require a PTY to work correctly with `exec()`. Examples follow:

#### exec():
```php
echo $ssh->exec('passwd');
```

Stalls. If you use a [callback function](#callbacks) or [setTimeout()](#settimeout) you'll see that it's outputting `(current) UNIX password:` and waiting for input that can't ever come.

#### exec() with PTY:
```php
$ssh->enablePTY();
$ssh->exec('passwd');
echo $ssh->read('password:');
$ssh->write("badpw\n");
echo $ssh->read('password unchanged');
```

Runs as one might expect. There may be a three second delay before the "password unchanged" dialog that's implemented by Linux to protect against brute force attacks (per "man pam_unix").

#### read() / write():
```php
echo $ssh->read('username@username:~$');
$ssh->write("passwd\n");
echo $ssh->read('password:');
$ssh->write("badpw\n");
echo $ssh->read('password unchanged');
```

Pretty much the same output as exec() with PTY.

### top

#### exec():
```php
echo $ssh->exec('top');
```

Outputs `TERM environment variable not set`.

#### exec() with PTY:
```php
$ssh->enablePTY();
$ssh->exec('top');
$ssh->setTimeout(5);
$ansi->appendString($ssh->read());
echo $ansi->getScreen();
```
See [ANSI Escape Codes](#ansi-escape-codes) for the output.

#### read() / write():
```php
$ansi->appendString($ssh->read('username@username:~$'));
$ssh->write("top\n");
$ssh->setTimeout(5);
$ansi->appendString($ssh->read());
echo $ansi->getScreen();
```
Pretty much the same output as exec() with PTY.

## Window Size

The default window size is 80x24 (80 columns, 24 rows).

The number of rows can be adjusted by calling `$ssh->setWindowRows()`. The number of columns can be adjusted by calling `$ssh->setWindowColumns()`. Both can be adjusted at the same time by calling `$ssh->setWindowSize(80, 24)`.

The number of rows and columns can be determined by calling `$ssh->getWindowRows()` and `$ssh->getWindowColumns()`, respectively.

## Multiple Channels

Let's say you ran a command with `$ssh->exec()` with a PTY open but that you also wanted to, simultaniously, run a command on an interactive shell. Nothing presented in the documentation, up to this point, really enables that, however, as of phpseclib v3.0.20, an additional parameter - `$channel` - has been added to `read()`, `write()` and `reset()`, so that now you can do this:

```php
$ssh->enablePTY();
$ssh->exec('sudo ls -latr');
$ssh->write("sudo ls -latr\n", SSH2::CHANNEL_SHELL);
$ssh->read('#[pP]assword[^:]*:|username@username:~\$#', SSH2::READ_REGEX, SSH2::CHANNEL_EXEC);
```
You can also, as of phpseclib v3.0.20, see the status of a channel by calling any of the following:

- `isShellOpen()`
- `isPTYOpen()`
- `isInteractiveChannelOpen($channel)`

Note that phpseclib does not currently let you have multiple shells or exec's with PTY or subsystems open.
