# `tmx-tmp`

TODO(dkorolev): Add `zsh` and `wget` the dotfile.
TODO(dkorolev): Add `y` to everything, including installs and the `ssh` command.

TODO(dkorolev): Turn this into a blog post. Add a link.

This is to be run inside Termux. A single copy-paste.

```
pkg upgrade
pkg install -y git openssh netcat-openbsd zbar curl

mkdir -p .ssh
chmod 700 .ssh

cat <<EOF >>.ssh/authorized_keys
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEeDzRfIMIC+OjsfVGsv9+q2RBeTGX2ZcrNNFa8n0i7o
EOF
chmod 600 .ssh/authorized_keys

cat <<EOF >.ssh/id_ed25519.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHL13Ujd8KlQsE/VJP6s5s934RRN+X/4H16pBqLmiV3J termux@tablet
EOF

cat <<EOF >.ssh/id_ed25519
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACBy9d1I3fCpULBP1ST+rObPd+EUTfl/+B9eqQai5oldyQAAAJAXLNzGFyzc
xgAAAAtzc2gtZWQyNTUxOQAAACBy9d1I3fCpULBP1ST+rObPd+EUTfl/+B9eqQai5oldyQ
AAAECu6NutqnO9viXJqGTkvlXwF0kmpb3C6GkcYzIJd0H0GnL13Ujd8KlQsE/VJP6s5s93
4RRN+X/4H16pBqLmiV3JAAAADXRlcm11eEB0YWJsZXQ=
-----END OPENSSH PRIVATE KEY-----
EOF

chmod 600 .ssh/id_ed25519.pub
chmod 600 .ssh/id_ed25519

git clone https://github.com/dkorolev/dotfiles
cp dotfiles/.zshrc .
echo zsh | chsh

# termux-setup-storage

sshd

# git clone https://github.com/dkorolev/current
# (cd current/bricks/strings; time test)

# passwd
# ipconfig

whoami

ssh -N -R 8022:localhost:8022 tmx@192.168.1.111

#ssh -R 8022:localhost:8022 tmx@192.168.1.111
```
