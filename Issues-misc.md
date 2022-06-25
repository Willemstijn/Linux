## Freqtrade installation problem on Fedora Linux

```
gcc: fatal error: cannot execute ‘cc1plus’: execvp: No such file or directory
      compilation terminated.
      error: command '/usr/bin/gcc' failed with exit code 1
      [end of output]
  
  note: This error originates from a subprocess, and is likely not a problem with pip.
  ERROR: Failed building wheel for py_find_1st
Failed to build blosc py_find_1st
ERROR: Could not build wheels for blosc, py_find_1st, which is required to install pyproject.toml-based projects
Failed installing dependencies
```

**Oplossing:**

Installeer: `sudo dnf install gcc-c++ python-devel`

## Dropbox: authentication is needed to run '/bin/sh' as the super user

If Dropbox finds files in its directory that aren't owned by your user, it will try to run a script with super user privileges to change the ownership of those files to your user.

Find the files in Dropbox that are not owned by you:

```
find ~/Dropbox/ ! -user $USER

/home/bas/Dropbox/CodeRepository/CandleHoarder/data/BNBUSDT.db
/home/bas/Dropbox/CodeRepository/CandleHoarder/data/SOLUSDT.db
/home/bas/Dropbox/CodeRepository/CandleHoarder/data/ADAUSDT.db

ls -alh

total 5,3M
-rw-r--r--. 1 root root 120K 15 jun 09:48 ADAUSDT.db
-rw-r--r--. 1 root root 120K 15 jun 10:06 ATOMUSDT.db
-rw-r--r--. 1 root root 120K 15 jun 09:47 BNBUSDT.db
-rw-r--r--. 1 root root 120K 15 jun 10:12 ENJUSDT.db
```

The safest option is to change the permissions yourself:

```
find ~/Dropbox/ ! -user $USER -exec sudo chown $USER:$USER "{}" \;
```

See also: https://askubuntu.com/questions/1062568/dropbox-asks-authentication-is-needed-to-run-usr-sh-as-the-super-user

##
