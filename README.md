<!-- Title-->
<p align="center">
  <h1 align="center">DartSSH 2</h1>
</p>

<!-- Badges-->
<p align="center">
  <a href="https://pub.dartlang.org/packages/dartssh2">
    <img src="https://img.shields.io/pub/v/dartssh2.svg">
  </a>
  <a href="https://github.com/TerminalStudio/dartssh2/actions/workflows/dart.yml">
    <img src="https://github.com/TerminalStudio/dartssh2/actions/workflows/dart.yml/badge.svg">
  </a>
  <a href="https://www.dartdocs.org/documentation/dartssh2/latest/">
    <img src="https://img.shields.io/badge/Docs-dartssh2-blue.svg">
  </a>
  <a href="https://ko-fi.com/F1F61K6BL">
    <img src="https://img.shields.io/badge/Buy%20Me%20a%20Coffee-F16061?style=flat&logo=buy-me-a-coffee&logoColor=white&labelColor=555555">
  </a>
</p>

<p align="center">
A SSH and SFTP client written in pure Dart, aiming to be feature-rich as well as easy to use.
</p>

> **dartssh2** is now a complete rewrite of [dartssh].

## ✨ Features

-  **Pure Dart**: Working with both Dart VM and Flutter.
-  **SSH Session**: Executing commands, spawning shells, setting environment variables, pseudo terminals, etc.
-  **Authentication**: Supports password, private key and interactive authentication method.
-  **Forwarding**: Supports local forwarding and remote forwarding.
-  **SFTP**: Supports all operations defined in [SFTPv3 protocol](https://datatracker.ietf.org/doc/html/draft-ietf-secsh-filexfer-02) including upload, download, list, link, remove, rename, etc.

## 🧪 Try

```sh
# Install the `dartssh` command.
dart pub global activate dartssh2

# Then use `dartssh` as regular `ssh` command.
dartssh user@example.com

# Example: execute a command on remote host.
dartssh user@example.com ls -al

# Example: connect to a non-standard port.
dartssh user@example.com:<port>

# Transfer files via SFTP.
dartsftp user@example.com
```

> If the `dartssh` command can't be found after installation, you might need to [set up your path](https://dart.dev/tools/pub/cmd/pub-global#running-a-script-from-your-path).

## 🚀 Quick start 

### Spawn a shell on remote host

```dart
final socket = await SSHSocket.connect('localhost', 22);

final client = SSHClient(
  socket,
  username: '<username>',
  onPasswordRequest: () => '<password>',
);

final shell = await client.shell();
stdout.addStream(shell.stdout);
stderr.addStream(shell.stderr);
stdin.cast<Uint8List>().listen(shell.write);

await shell.done;
client.close();
```

### Execute a command on remote host


```dart
final uptime = await client.run('uptime');
print(utf8.decode(uptime));
```

### Forward connections on local port 8080 to the server

```dart
final serverSocket = await ServerSocket.bind('localhost', 8080);
await for (final socket in serverSocket) {
  final forward = await client.forwardLocal('httpbin.org', 80);
  forward.stream.cast<List<int>>().pipe(socket);
  socket.pipe(forward.sink);
}
```

### Forward connections to port 2222 on the server to local port 22

```dart
final forward = await client.forwardRemote(port: 2222);

if (forward == null) {
  print('Failed to forward remote port');
  return;
}

await for (final connection in forward.connections) {
  final socket = await Socket.connect('localhost', 22);
  connection.stream.cast<List<int>>().pipe(socket);
  socket.pipe(connection.sink);
}
```

### Authenticate with public keys

```dart
final client = SSHClient(
  socket,
  username: '<username>',
  identities: [
    // A single private key file may contain multiple keys.
    ...SSHKeyPair.fromPem(await File('path/to/id_rsa').readAsString())
  ],
);
```

### Use encrypted PEM files
```dart
// Test whether the private key is encrypted.
final encrypted = SSHKeyPair.isEncrypted(await File('path/to/id_rsa').readAsString());
print(encrypted);

// If the private key is encrypted, you need to provide the passphrase.
final keys = SSHKeyPair.fromPem('<pem text>', '<passphrase>');
print(keys);
```

Decrypt PEM file with [`compute`](https://api.flutter.dev/flutter/foundation/compute-constant.html) in Flutter

```dart
List<SSHKeyPair> decryptKeyPairs(List<String> args) {
  return SSHKeyPair.fromPem(args[0], args[1]);
}

final keypairs = await compute(decryptKeyPairs, ['<pem text>', '<passphrase>']);
```

### Get the version of SSH server

```dart
await client.authenticated;
print(client.remoteVersion); // SSH-2.0-OpenSSH_7.4p1
```

## SFTP

### List remote directory

```dart
import 'package:dartssh2/dartssh2.dart';

void main(List<String> args) async {
  final socket = await SSHSocket.connect('localhost', 22);

  final client = SSHClient(
    socket,
    username: 'root',
    username: '<username>',
    onPasswordRequest: () => '<password>',
  );

  final sftp = await client.sftp();
  final items = await sftp.listdir('/');
  for (final item in items) {
    print(item.longname);
  }

  client.close();
  await client.done;
}
```

### Read remote file

```dart
final sftp = await client.sftp();
final file = await sftp.open('/etc/passwd');
final content = await file.readBytes();
print(latin1.decode(content));
```

### Directory operations
```dart
final sftp = await client.sftp();
await sftp.mkdir('/path/to/dir');
await sftp.rmdir('/path/to/dir');
```
### Get/Set attributes from/to remote file/directory
```dart
await sftp.stat('/path/to/file');
await sftp.setStat(
  '/path/to/file',
  SftpFileAttrs(mode: SftpFileMode(userRead: true)),
);
```

### Create a link
```dart
final sftp = await client.sftp();
sftp.link('/from', '/to');
```

## 🪜 Example

### SSH client:

- [example/example.dart](https://github.com/TerminalStudio/dartssh2/blob/master/example/example.dart)
- [example/forward_local.dart](https://github.com/TerminalStudio/dartssh2/blob/master/example/forward_local.dart)
- [example/forward_remote.dart](https://github.com/TerminalStudio/dartssh2/blob/master/example/forward_remote.dart)
- [example/pubkey.dart](https://github.com/TerminalStudio/dartssh2/blob/master/example/pubkey.dart)
- [example/shell.dart](https://github.com/TerminalStudio/dartssh2/blob/master/example/shell.dart)

### SFTP:
- [example/sftp_read.dart](https://github.com/TerminalStudio/dartssh2/blob/master/example/sftp_read.dart)
- [example/sftp_list.dart](https://github.com/TerminalStudio/dartssh2/blob/master/example/sftp_list.dart)
- [example/sftp_stat.dart](https://github.com/TerminalStudio/dartssh2/blob/master/example/sftp_stat.dart)



## 🔐 Supported algorithms

**Host key**: 
- `ssh-rsa`
- `rsa-sha2-[256|512]`
- `ecdsa-sha2-nistp[256|384|521]`
- `ssh-ed25519`

**Key exchange**: 
- `curve25519-sha256`
- `ecdh-sha2-nistp[256|384|521] `
- `diffie-hellman-group-exchange-sha[1|256]`
- `diffie-hellman-group14-sha[1|256]`
- `diffie-hellman-group1-sha1 `
  
**Cipher**: 
- `aes[128|192|256]-ctr`
- `aes[128|192|256]-cbc`

**Integrity**: 
- `hmac-md5`
- `hmac-sha1`
- `hmac-sha2-[256|512]`
  
## ⏳ Roadmap

- [x] Fix broken tests
- [x] Sound null safety
- [x] Redesign API to allow starting multiple sessions.
- [x] Full SFTP
- [ ] Server

## References

- [`RFC 4250`](https://datatracker.ietf.org/doc/html/rfc4250) The Secure Shell (SSH) Protocol Assigned Numbers
- [`RFC 4251`](https://datatracker.ietf.org/doc/html/rfc4251) The Secure Shell (SSH) Protocol Architecture
- [`RFC 4252`](https://datatracker.ietf.org/doc/html/rfc4252) The Secure Shell (SSH) Authentication Protocol
- [`RFC 4253`](https://datatracker.ietf.org/doc/html/rfc4253) The Secure Shell (SSH) Transport Layer Protocol
- [`RFC 4254`](https://datatracker.ietf.org/doc/html/rfc4254) The Secure Shell (SSH) Connection Protocol
- [`RFC 4255`](https://datatracker.ietf.org/doc/html/rfc4255) Using DNS to Securely Publish Secure Shell (SSH) Key Fingerprints
- [`RFC 4256`](https://datatracker.ietf.org/doc/html/rfc4256) Generic Message Exchange Authentication for the Secure Shell Protocol (SSH)
- [`RFC 4419`](https://datatracker.ietf.org/doc/html/rfc4419) Diffie-Hellman Group Exchange for the Secure Shell (SSH) Transport Layer Protocol
- [`RFC 4716`](https://datatracker.ietf.org/doc/html/rfc4716) The Secure Shell (SSH) Public Key File Format
- [`RFC 5656`](https://datatracker.ietf.org/doc/html/rfc5656) Elliptic Curve Algorithm Integration in the Secure Shell Transport Layer
- [`RFC 8332`](https://datatracker.ietf.org/doc/html/rfc8332) Use of RSA Keys with SHA-256 and SHA-512 in the Secure Shell (SSH) Protocol
- [`RFC 8731`](https://datatracker.ietf.org/doc/html/rfc8731) Secure Shell (SSH) Key Exchange Method Using Curve25519 and Curve448
- [`draft-miller-ssh-agent-03`](https://datatracker.ietf.org/doc/html/draft-miller-ssh-agent-03) SSH Agent Protocol
- [`draft-ietf-secsh-filexfer-02`](https://datatracker.ietf.org/doc/html/draft-ietf-secsh-filexfer-02) SSH File Transfer Protocol
- [`draft-dbider-sha2-mac-for-ssh-06`](https://datatracker.ietf.org/doc/html/draft-dbider-sha2-mac-for-ssh-06) SHA-2 Data Integrity Verification for the Secure Shell (SSH) Transport Layer Protocol

## Credits

https://github.com/GreenAppers/dartssh by GreenAppers

## License

dartssh is released under the terms of the MIT license. See [LICENSE](LICENSE).

[dartssh]: https://github.com/GreenAppers/dartssh