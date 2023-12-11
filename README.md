# `tmx-tmp`

TODO(dkorolev): Turn this into a blog post. Add a link.

This is to be run inside Termux. A single copy-paste.

```
yes | pkg -y upgrade
yes | pkg install -y git zsh screen vim-python curl wget openssh openssl-tool netcat-openbsd libqrencode zbar

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

#git clone https://github.com/dkorolev/dotfiles
#cp dotfiles/.zshrc .

for i in .zshrc .vimrc .screenrc .inputrc ; do
  https://raw.githubusercontent.com/dkorolev/dotfiles/main/$i
done

export PATH=$PATH:$PWD
echo 'export PATH=$PATH:$PWD' >>~/.zshrc

export SECRET_TMX_PASSWORD=lopata
echo 'export SECRET_TMX_PASSWORD=lopata' >>~/.zshrc

chsh -s zsh

sshd

# git clone https://github.com/dkorolev/current
# (cd current/bricks/strings; time test)

# passwd
# ipconfig

echo '(echo -n 'tmx://'; echo "TMXUSER:$(whoami)" | openssl aes-256-cbc -pbkdf2 -a -salt -pass pass:$SECRET_TMX_PASSWORD) | qrencode -t ascii | sed "s/\#/█/g"' >~/w

chmod +x ~/w

termux-setup-storage

whoami

ssh -o StrictHostKeyChecking=accept-new -N -R 8022:localhost:8022 tmx@192.168.1.111
```
