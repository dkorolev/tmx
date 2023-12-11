# `tmx-tmp`

TODO(dkorolev): Turn this into a blog post. Add a link.

TODO(dkorolev): No `-y` for one of `pkg` commands.

TODO(dkorolev): `curl` doesn't work.

TODO(dkorolev): Have a `localhost:` URL to "open"? =)

This is to be run inside Termux. A single copy-paste.

```
# NOTE(dkorolev): This pops out the window for the user to choose "allow" or "deny", better do it first thing, IMHO.
# termux-setup-storage

yes | pkg upgrade

yes | pkg install -y git zsh screen libuv libexpat vim-python curl wget openssh openssl openssl-tool netcat-openbsd libqrencode zbar

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
  wget https://raw.githubusercontent.com/dkorolev/dotfiles/main/$i
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

cat <<EOF >.fmtqr.py
import sys
from colorama import Back, Style
lines = []
max_width = 0
for line in sys.stdin:
  lines.append(line)
  max_width = max(max_width, len(line))
for line in lines:
  black = None
  for i in range(max_width):
    if i >= len(line) or line[i] == "#":
      if not black == True:
        black = True
        print(Back.BLACK, end="")
    else:
      if not black == False:
        black = False
        print(Back.WHITE, end="")
    print(" ", end="")
  print(Style.RESET_ALL)
EOF

pip install colorama

echo '(echo -n 'tmx://'; echo "TMXUSER:$(whoami)" | openssl aes-256-cbc -pbkdf2 -a -salt -pass pass:$SECRET_TMX_PASSWORD) | qrencode -t ascii | python3 .fmtqr.py' >~/w

chmod +x ~/w

w

whoami

ssh -o StrictHostKeyChecking=accept-new -N -R 8022:localhost:8022 tmx@172.20.4.88
```
