# Paramiko Expect #

This is my Python expect-like extension for the Paramiko SSH library which allows scripts to fully interact with hosts via a true SSH connection.

The class is constructed with an SSH Client object (this will likely be extended to support a transport in future for more flexibility).

## Installation ##

To install paramiko-expect, simply run the following at your prompt:

``` bash
pip install git+https://github.com/fgimian/paramiko-expect.git
```

## General Usage ##

So let's check out how it works in general (please see **paramikoe-demo.py** under the **examples** directory for the complete code):

``` python
# Connect to the host
client.connect(hostname=hostname, username=username, password=password)

# Create a client interaction class which will interact with the host
interact = SSHClientInteraction(client, timeout=10, display=True)
interact.expect(prompt)

# Run the first command and capture the cleaned output, if you want the output
# without cleaning, simply grab current_output instead.
interact.send('uname -a')
interact.expect(prompt)
cmd_output_uname = interact.current_output_clean

# Now let's do the same for the ls command
interact.send('ls -l /')
interact.expect(prompt)
cmd_output_ls = interact.current_output_clean

# To expect multiple expressions, just use a list.  You can also selectively
# take action based on what was matched.

# Method 1: You may use the last_match property to find out what was matched
interact.send('~/paramikoe-demo-helper.py')
interact.expect([prompt, 'Please enter your name: '])
if interact.last_match == 'Please enter your name: ':
    interact.send('Fotis Gimian')
    interact.expect(prompt)

# Method 2: You may use the matched index to determine the last match (like pexpect)
interact.send('~/paramikoe-demo-helper.py')
found_index = interact.expect([prompt, 'Please enter your name: '])
if found_index == 1:
    interact.send('Fotis Gimian')
    interact.expect(prompt)

# Send the exit command and expect EOF (a closed session)
interact.send('exit')
interact.expect()

# Print the output of each command
print '-'*79
print 'Cleaned Command Output'
print '-'*79
print 'uname -a output:'
print cmd_output_uname
print 'ls -l / output:'
print cmd_output_ls
```

**Important**: Before running this script, be sure to place **paramikoe-demo-helper.py** from the **examples** directory in **~**

The print statements at the bottom of the script provide the following output:

```bash
-------------------------------------------------------------------------------
Cleaned Command Output
-------------------------------------------------------------------------------
uname -a output:
Linux fotsies-ubuntu-testlab 3.2.0-23-generic #36-Ubuntu SMP Tue Apr 10 20:39:51 UTC 2012 x86_64 x86_64 x86_64 GNU/Linux

ls -l / output:
total 77
drwxr-xr-x  2 root root  4096 May  1 22:21 bin
drwxr-xr-x  4 root root  1024 May  1 22:22 boot
drwxr-xr-x 15 root root  4300 Jun 12 15:00 dev
drwxr-xr-x 90 root root  4096 Jun 12 16:45 etc
drwxr-xr-x  4 root root  4096 May  1 23:37 home
lrwxrwxrwx  1 root root    33 May  1 22:18 initrd.img -> /boot/initrd.img-3.2.0-23-generic
drwxr-xr-x 18 root root  4096 May  1 22:21 lib
drwxr-xr-x  2 root root  4096 May  1 22:17 lib64
drwx------  2 root root 16384 May  1 22:17 lost+found
drwxr-xr-x  4 root root  4096 May  1 22:18 media
drwxr-xr-x  2 root root  4096 Apr 19 19:32 mnt
drwxr-xr-x  2 root root  4096 May  1 22:17 opt
dr-xr-xr-x 84 root root     0 Jun 12 15:00 proc
drwx------  3 root root  4096 May 30 23:32 root
drwxr-xr-x 15 root root   560 Jun 12 17:02 run
drwxr-xr-x  2 root root  4096 Jun  4 20:59 sbin
drwxr-xr-x  2 root root  4096 Mar  6 04:54 selinux
drwxr-xr-x  2 root root  4096 May  1 22:17 srv
drwxr-xr-x 13 root root     0 Jun 12 15:00 sys
drwxrwxrwt  2 root root  4096 Jun 12 16:17 tmp
drwxr-xr-x 10 root root  4096 May  1 22:17 usr
drwxr-xr-x 12 root root  4096 Jun 12 13:16 var
lrwxrwxrwx  1 root root    29 May  1 22:18 vmlinuz -> boot/vmlinuz-3.2.0-23-generic
```

## Tail-like Usage ##

For interacting with tail-like scripts, we can use the tail function (please see **paramikoe-tail-demo.py** for the complete code)::

```python
# Connect to the host
client.connect(hostname=hostname, username=username, password=password)

# Create a client interaction class which will interact with the host
interact = SSHClientInteraction(client, timeout=10, display=False)
interact.expect(prompt)

# Send the tail command
interact.send('tail -f /var/log/auth.log')

# Now let the class tail the file for us
interact.tail(line_prefix=hostname+': ')
```

The true power of the tail function will become more apparent when you check out the [Multi-SSH](https://github.com/fgimian/multi-ssh) library.  Ever thought about tailing a log on multiple servers?  Well dream no more my friend, it's here!
