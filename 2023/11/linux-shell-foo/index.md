# Linux Shell Foo

## Find

Get the most recently modified file from a directory:
```shell
find /path/to/target/dir -type f -exec ls -t1 {} + | head -1
```

Find the 10 largest files of a directory and its sub directories recursively:
```shell
find / -type f -printf '%s %p\n' | sort -nr | head -10
```

Find all files between 10Mb and 20Mb
```shell
find / -type f -size +10M -size -20M
```

Find all files created after  `00:00:00 31.12.2020`
```shell
find / -type f -newermt '2020-12-31'
```

Find all files created before `00:00:00 31.12.2020`
```shell
find / -type f ! -newermt '2020-12-31'
```

Find and delete all files with the name `special.txt` and delete them
```shell
find / -type f -regex ".*\/special\.txt" -delete
```

## Locate

The locate command does not need to walk the whole file system to locate a file you search for. It uses a database which can be updated by the following command:
```shell
sudo updatedb
```

After that you can use search strings with wildcards to locate specific files like this:
```bash
locate *.conf
```

One drawback, in comparison to find, is that it does not provide as many search filters.

## column

To view multi line output formatted as table you can use the `column` command with the `-t` flag.
For example, we want to view the inode number and the name of every file in a directory. Without column the output looks like this:
```shell
ls -i
876080 g5.txt  876077 h1.txt  876078 h2.txt  876079 h3.txt  876081 h4.txt  876076 specific.special
```

we can pipe it into `column -t` to format it:
```shell
ls -i | column -t
876080  g5.txt
876077  h1.txt
876078  h2.txt
876079  h3.txt
876081  h4.txt
876076  specific.special
```

## awk

Using `netstat` list all services listening on all interfaces (0.0.0.0) for IPv4:
```shell
netstat -luntp | grep "LISTEN" | awk '$4 ~ /0\.0\.0\.0*/' | column -t
```


## grep

### regex

To use the patterns run `grep` with the `-E` flag for extended regex

|Operator|Description|
|----------|------------|
| (a)      |The round brackets are used to group parts of a regex. Within the brackets, you can define further patterns which should be processed together.|
| [a-z]   | The square brackets are used to define character classes. Inside the brackets, you can specify a list of characters to search for. |
| {1,10} | The curly brackets are used to define quantifiers. Inside the brackets, you can specify a number or a range that indicates how often a previous pattern should be repeated. |
| \| | Also called the OR operator and shows results when one of the two expressions matches |
| .* | Also called the AND operator and displayed results only if both expressions match |


## systemctl

Nice looking list of all services
```shell
systemctl list-units --type=service
```

Add Service to SysV to start automatically
```shell
systemctl enable <service-name>
```


### Background a process

It is possible to suspend process execution and send it to the background by pressing `Ctr-z` to send a `SIGSTP` signal to the kernel which suspends the process.
We can resume execution by running the `fg` command which puts the process back to the foreground.
This can come in handy if you are in your text editor and just want to quickly switch out of it, run a command and then switch back to it.

You can also start a process which will run in the background and send its result to your shell as soon as it exits by adding a `&` to the end:

```shell
ping -c 10 https://www.google.com &
```


### rsync

Transfer files from a directory, keeping file attributes like permissions, using compression for quicker transmission and ssh for encrypted transport:

```shell
rsync -az -e ssh /path/to/local/dir user@server:/path/of/remote/dir
```


### mount

List all mounted drives:
```shell
mount
```

Mount a Usb stick to the local filesystem to the directory `/mnt/usb`:
```shell
mount /dev/sdb1 /mnt/usb
```

Unmount the Usb stick:
```shell
unmount /mnt/usb
```


## ifconfig

Assign net mask to a network interface

```shell
sudo ifconfig eth0 netmask 255.255.255.0
```

Assign an IP address to a network interface

```shell
sudo ifconfig eth0 192.168.10.2
```

## route

Set the address default gateway for a network interface to be used to send traffic outside the current network the interface is connected to

```shell
sudo route add default gw 192.68.10.1 eth0
```

## tee

If you want to redirect the output of a command in a sequence of commands connected by pipes you can use `tee` to redirect output to a file in between chained commands:

```shell
whois 10.10.10.1 | tee out.txt | grep "NetRange\|CIDR"
```

If you want to run `tee` as part of a loop and append to a file use the `-a` flag

```shell
for ip in $addrs
do
	whois "$ip" | tee -a out.txt | grep "NetRange\|CIDR"
done
```

## Keyboard Shortcuts

- `CTRL-a` move the cursor to the beginning of the current line
- `CTRL-e` move the cursor to the end of the current line
- `CTRL-<left-arrow|right-arrow>` jump to the beginning of the current/previous word
- `ALT-B` jump back one word
- `ALT-F` jump one word forward
- `CTRL-R` search through the command history
- `CTRL-u` erase everything from the current position of the cursor to the beginning if the line
- `CTRL-k` erase everything from the current position of the cursor to the end of the line
- `CTRL-w` erase the word preceding the cursor position

