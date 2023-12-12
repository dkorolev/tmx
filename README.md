# `tmx-tmp`

TODO(dkorolev): Turn this into a blog post. Add a link.

TODO(dkorolev): Add the link to the Termux `.apk` link I'm using.

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

cat <<EOF >fmtqr.py
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

# NOTE(dkorolev): Historically, `w` is a one-char command so that it's easy to type from mobile keyboard.
echo '(echo -n 'tmx://'; echo "TMXUSER:$(whoami)" | openssl aes-256-cbc -pbkdf2 -a -salt -pass pass:$SECRET_TMX_PASSWORD) | qrencode -t ascii | python3 fmtqr.py' >~/w

chmod +x ~/w

# w

cat <<EOF >t
#!/bin/bash
#
# Opens an tunnel and forwards port 8022.

if [ "\$1" != "" ] ; then
  echo "Decyphering \$1"
  echo "\$1"
  echo -n "\$1" | sed 's/_/\//g'
  if HOST=\$(echo \$1 | sed 's/_/\//g' | openssl aes-256-cbc -d -a -pbkdf2 -pass pass:\$SECRET_TMX_PASSWORD) ; then
    echo "Opening tunnel to \$HOST"
    ssh -N -R 8022:localhost:8022 tmx@\$HOST &
  else
    echo 'Can not decypher, ensure the \$SECRET_TMX_PASSWORD is correct on both ends.'
  fi
else
  echo "Need host."
fi
EOF
chmod +x t

cat <<EOF >s.py
from http.server import BaseHTTPRequestHandler, HTTPServer

import os

class SimpleHTTPRequestHandler(BaseHTTPRequestHandler):
  def do_GET(self):
    fields = self.path.strip().split('/')
    if len(fields) >= 3 and fields[-3] == "tmx" and fields[-2] == "rvp":
      self.send_response(200)
      self.send_header('Content-type', 'text/plain')
      self.end_headers()
      self.wfile.write(b"OK, tunnel!\n")
      print(f"DEBUG: ~/t {fields[-1]}")
      os.system(f"~/t {fields[-1]}")
    else:
      self.send_response(500)
      self.send_header('Content-type', 'text/plain')
      self.end_headers()
      self.wfile.write(b"My responses are limited. You must ask the right questions.\n")

def run(server_class=HTTPServer, handler_class=SimpleHTTPRequestHandler):
  server_address = ('', 8088)
  httpd = server_class(server_address, handler_class)
  httpd.serve_forever()

if __name__ == '__main__':
  run()
EOF

python3 s.py &

# whoami
# ssh -o StrictHostKeyChecking=accept-new -N -R 8022:localhost:8022 tmx@172.20.4.88

echo
echo 'Everything is up and running. Now:'
echo
echo '1) Run the magic on the laptop, gen the QR code.'
echo '2) Scan this QR code from the tablet, open the URL.'
echo '3) The tunnel will open. Can `ssh -p 8022 tmx@localhost` now.'
```

Now, on the host machine.

(I am skipping the user creation part for now. -- D.K.)

```
# Make sure the same `fmtqr.py` is available on the host machine.
[ -s ~/fmtqr.py ] || cat <<EOF >~/fmtqr.py
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

# Generate the QR code. TODO(dkorolev): Explain this in detail!
echo http://localhost:8088/tmx/rvp/$(ip route get 8.8.8.8 | awk '{printf "%s\n", $7}' | openssl aes-256-cbc -pbkdf2 -a -salt -pass pass:$SECRET_TMX_PASSWORD | sed 's/\//_/g') | qrencode -t ascii | python3 ~/fmtqr.py
```

