[![Build Status](https://travis-ci.org/mkishere/sshsyrup.svg?branch=master)](http://travis-ci.org/mkishere/sshsyrup) [![Build status](https://ci.appveyor.com/api/projects/status/iy271guyn7ig81yn/branch/master?svg=true)](https://ci.appveyor.com/project/mkishere/sshsyrup/branch/master)
# Syrup
A SSH honeypot with rich features written in Go

### Features
- SSH self-defined accounts and passwords, also allow any logins
- Fake shell. Records shell sessions and upload to [asciinema.org](https://asciinema.org) (Or, if you wish, can log as [UML-compatible](http://user-mode-linux.sourceforge.net/old/tty_logging.html) format)
- Virtual Filesystem for browsing and fooling intruder
- Logs key signature to log
- Log in JSON format for easy parsing
- Record local and remote host when client attempt to create port redirection
- Structure allows [extending command sets](https://github.com/mkishere/sshsyrup/wiki/Writing-new-commands) with ease

### See Recorded Session in Action!
[![asciicast](https://asciinema.org/a/rgr1KyY1Xn21bXIDMKL9fkGD0.png)](https://asciinema.org/a/rgr1KyY1Xn21bXIDMKL9fkGD0)

### Requirements
- Linux, Mac or Windows (I've only tested in Windows and WSL, suppose the other 2 platforms should work as expected)
- Go 1.9 or up (For building)
- [dep](https://github.com/golang/dep) (For building)

### Download
You may find the pre-build packages for various platform on the [release](https://github.com/mkishere/sshsyrup/releases) tab. If you find the platform you need is not on the list, you can follow the building procedure in the next section.

### Building
Run the following command in shell to get latest package and build it
```
go get -u github.com/mkishere/sshsyrup
cd ~/go/src/github.com/mkishere/sshsyrup
dep ensure
go build -ldflags "-s -w" -o sshsyrup ./cmd/syrup
go build -ldflags "-s -w" -o createfs ./cmd/createfs
```

### Setting up for the first run
* Create and modify config.json. Here are the sample configuration (minimal setup)
```json
{
    "server.addr": "0.0.0.0",
    "server.port": 22,
    "server.allowRandomUser": false
}
```
* Prepare the virtual filesystem image by downloading the filesystem.zip from master branch or create your own by running
```
./createfs -p / -o filesystem.zip
```
Since we'll need to read every file from the directory, it will take some time to load.

_For Windows, since there are no user/group information, the file/directory owner will always be root._

* Prepare user and passwd file
Put _passwd_ and _group_ file in the same directory as config.json. The format of both files are the same as their [real-life counterpart](http://www.linfo.org/etc_passwd.html) in _/etc_, except that passwd also stores the password in the second field of each line, and asterisk(*) in password field can be used to denote matching any password.
* Generate SSH private key and renamed as id_rsa and put it in the same directory
```
ssh-keygen -t rsa
```
* Start the server
```
./sshsyrup
```
### Config params
Parameter Name | Description
-------------- | -----------
server.addr | Server address the honeypot will listen to. Default to _0.0.0.0_
server.port | Port number the honeypot will listen to. Default to _2222_
server.allowRandomUser | Allow user to login with any user names. Default to _false_
server.ident | Server identification string. Default to _SSH-2.0-OpenSSH_6.8p1_
server.maxConnections | Max connections allowed simutaneously. Default to _10_
server.sessionLogFmt | Terminal session log format. Can be _asciinema_ or _uml_. Default _asciinema_
asciinema.logfileprefix | Prefix to write asciinema compatible terminal replay files. Default to _test_
asciinema.apiEndpoint | The service endpoint for the replay to upload. Default to _https://asciinema.org_
asciinema.apiKey | The token ID generated by the `asciinema` command. If empty the file will not be uploaded to asciinema.org. You may find the key in ~/.config/asciinema/config

### Logging
By default Syrup will create a logging file in _logs/_ directory with file name _yyyymmdd.log_ in JSON format.

When available, the log will provide the information in following fields:

Field Name | Description
---------- | -----------
clientStr | Client identification string
sessionId | Session ID in base64
srcIP | Client IP
time | Log time
user | User account client used to login
password | Password used by client to login, only available when logging in
cmd | The command user type in shell

Also, each terminal session (the shell) will be logged into a separate file under logs/sessions in [asciinema v2 format](https://github.com/asciinema/asciinema/blob/develop/doc/asciicast-v2.md).

### Contributing
Feel free to submit feature request/bug report via the GitHub issue tracker.

### TODO
- Minimal set of POSIX commands/utilites
- Mock SFTP/SCP/Port forwarding
