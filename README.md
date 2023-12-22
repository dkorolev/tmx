# `tmx`

TODO(dkorolev): Link to the blog post.

Here are quick links to the respective commands.

Github has the feature to copy the contents of multiline code blocks to the clipboard by clicking or tapping in the upper right corner of the respective code block. This comes in very handy from the mobile device.

* Host machine `tmx` user setup: [subsection](#host-machine-setup).
* If using `adb`, install Termux from the host machine: [subsection](#install-termux-via-adb).
* Termux commands to copy-paste: [subsection](#commands-inside-termux).
* Host machine connection QR code generation: [subsection](#host-machine-generate-qr-code).
* Open the connection via the reverse SSH tunnel: [subsection](#connect).

## Instructions

* Read the code blocks below.
  * I've made it to be as trustworthy as possible, but, hey, it's the Internet.
  * The code is self-contained except my `dotfiles`. So, at least in theory, there is a security hazard present. Forewarned forearmed.
* Open this page.
  * Both on the host machine and on the Android device.
* Create a local `tmx` user on the host machine.
  * The first link above.
* Install Termux on the Android device.
  * The second link above.
  * Can install from the device itself, or via `adb`.
* On the Anroid device, copy the long Termux command into clipboard.
  * The third link above.
  * Github conveniently offers a "copy" button in the upper right corner of the large code block.
* Open Termux on the Android device and run this command.
  * You will need to accept notifications first.
  * Long tap, wait until the context menu appears, press "Paste", press "Enter"
  * Can also do this comfortably via `scrcpy`.
  * The command runs for about ~2.5 minutes in goog enough Internet (on my Samsung Galaxy S7+; a bit longer on Pixel 3a).
* Generate the QR code on the host machine.
  * The fourth link above.
* Scan this QR code from the Android device.
  * It should suggest to open a long link starting with `http://localhost`. Open it.
  * On LineageOS, the camera app does not scan QR codes. Use the QR code scanner from "settings".
* As the page opens successfully, the reverse SSH tunnel will be open from the Android device to the host machine.
  * Connect to it from the host machine following the fifth and final link above.

## Install Termux via ADB

[Up](#tmx)

This installs Termux [v0.118.0](https://github.com/termux/termux-app/releases/tag/v0.118.0) via `adb` from the hots machine.

Alternatively, open the above link from the Android device, download the APK, and install it manually, by tapping "Install" and enabling the Android browser to install custom APKs.

```
[ -f termux-app_v0.118.0+github-debug_arm64-v8a.apk ] || wget https://github.com/termux/termux-app/releases/download/v0.118.0/termux-app_v0.118.0+github-debug_arm64-v8a.apk
adb uninstall com.termux
adb install termux-app_v0.118.0+github-debug_arm64-v8a.apk
adb shell monkey -p com.termux -c android.intent.category.LAUNCHER 1
```

## Host machine setup

[Up](#tmx)

Create the `tmx` user used for reverse tunneling.

```
sudo useradd -m tmx

sudo mkdir -p /home/tmx/.ssh
sudo touch /home/tmx/.ssh/authorized_keys
sudo cat /home/tmx/.ssh/authorized_keys

sudo bash -c 'echo ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHL13Ujd8KlQsE/VJP6s5s934RRN+X/4H16pBqLmiV3J termux@tablet >>/home/tmx/.ssh/authorized_keys'

sudo chown -R tmx: /home/tmx/.ssh
sudo chmod 700 /home/tmx/.ssh
sudo chmod 600 /home/tmx/.ssh/authorized_keys
```

FTR, to revert:

```
sudo userdel tmx
sudo rm -rf /home/tmx
```

## Commands inside Termux

[Up](#tmx)

After you "copy" these commands, if you're in `scrcpy`, long-press the main mouse button inside Termux for the menu with "paste" to appear.

```
# NOTE(dkorolev): This pops out the window for the user to choose "allow" or "deny", better do it first thing, IMHO.
# termux-setup-storage

yes | pkg upgrade

yes | pkg install -y git zsh screen libuv libexpat vim-python curl wget openssh openssl openssl-tool netcat-openbsd libqrencode zbar

mkdir -p .ssh
chmod 700 .ssh

cat <<EOF >>.ssh/authorized_keys
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEeDzRfIMIC+OjsfVGsv9+q2RBeTGX2ZcrNNFa8n0i7o
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHL13Ujd8KlQsE/VJP6s5s934RRN+X/4H16pBqLmiV3J termux@tablet
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

for i in .zshrc .vimrc .screenrc .inputrc fmtqr.py ; do
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

pip install colorama
chmod +x fmtqr.py

# NOTE(dkorolev): Historically, `w` is a one-char command so that it's easy to type from mobile keyboard.
# TODO(dkorolev): Remove everything under `scripts/`, and perhaps use an `alias` for the line-liner instead!

echo '(echo -n 'tmx://'; echo "TMXUSER:$(whoami)" | openssl aes-256-cbc -pbkdf2 -a -salt -pass pass:$SECRET_TMX_PASSWORD) | qrencode -t ascii | python3 fmtqr.py' >~/w
chmod +x ~/w

# w

echo '''#!/bin/bash
#
# Opens an tunnel and forwards port 8022.
if [ "$1" != "" ] ; then
  echo "Decyphering $1"
  echo "$1"
  echo -n "$1" | sed "s/_/\//g"
  if HOST=$(echo $1 | sed "s/_/\//g" | openssl aes-256-cbc -d -a -pbkdf2 -pass pass:$SECRET_TMX_PASSWORD) ; then
    echo "Opening tunnel to $HOST"
    ssh -o StrictHostKeyChecking=accept-new -N -R 8022:localhost:8022 tmx@$HOST >/dev/null 2>&1 &
  else
    echo "Can not decypher, ensure the \$SECRET_TMX_PASSWORD is correct on both ends."
  fi
else
  echo "Need host."
fi
''' >open_tunnel.sh
chmod +x open_tunnel.sh

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
      self.wfile.write(b"ssh -p 8022 tmx@localhost  # Should work from the host machine now.\n")
      print(f"DEBUG: ~/open_tunnel.sh {fields[-1]}")
      os.system(f"~/open_tunnel.sh {fields[-1]}")
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

python3 s.py >/dev/null 2>&1 &

# whoami
# ssh -o StrictHostKeyChecking=accept-new -N -R 8022:localhost:8022 tmx@172.20.4.88

echo
echo 'Everything is up and running. Now:'
echo
echo '1) Run the magic on the laptop, gen the QR code.'
echo '2) Scan this QR code from the tablet, open the URL. This opens the tunnel.'
echo '3) Can `ssh -p 8022 tmx@localhost` from the host machine now.'
```

## Host machine generate QR code

[Up](#tmx)

```
[ -s ~/fmtqr.py ] || ((cd; wget https://raw.githubusercontent.com/dkorolev/dotfiles/main/fmtqr.py);chmod +x ~/fmtqr.py)

export SECRET_TMX_PASSWORD=lopata
echo && echo http://localhost:8088/tmx/rvp/$(ip route get 8.8.8.8 | awk '{printf "%s\n", $7}' | openssl aes-256-cbc -pbkdf2 -a -salt -pass pass:$SECRET_TMX_PASSWORD | sed 's/\//_/g') | qrencode -t ascii | python3 ~/fmtqr.py && echo && echo 'Run `ssh -p 8022 tmx@localhost` after scanning this QR code from the Android device.'
```

After scanning this QR code from the Android device with all the above steps completed, the reverse SSH tunnel will open.

## Connect

[Up](#tmx)

From the host machine, effectively, just `ssh -p 8022 foo@localhost`. 

```
ssh-keygen -f "/home/ubuntu/.ssh/known_hosts" -R "[localhost]:8022"
ssh -o StrictHostKeyChecking=accept-new -p 8022 foo@localhost
```

## Further Geeking

Relatively straightforward:

* Clean up the scripts to have zero extra dependencies, so that it feels secure enough to “just run” it.
* Have the logic that runs on the host machine inform the user that the tunnel was successfully open.
* Make “connection codes” one-time-use. And limit their lifetime to several seconds.
* Daemonize the HTTP server and the tunnel-opening services properly within Termux.
* Add a 403 redirect so that the page open in a browser on Android after scanning the QR code is not auto-reloading upon browser re-open.
* Add a note on how many seconds did the setup take.
* Add a logfile for remote tunnels open and closed. Guard with `flock`.

Nice-to-have:

* Given Termux is open source, have a custom build of it that incorporates the above. Done right, this gets rid of the need to run any commands.
* In fact, have this custom build produce the `.apk` from a possibly private Github repo/fork with a pre-built Github action. So that it could, among other things, have my own, secret, private key.
* When it is a custom app, register the URL schema for some `tmx://` URLs, for more native integration. So that the logic to open the reverse SSH tunnel is run automatically. In fact, the very on-device terminal shell can be hidden by default, as it is largely unused.

Long-term:

* Have all Android devices all around the home and/or office turn into a distributed compilation cluster. And a build artifacts cache.
* Complete the app with a DAX-like setup, so that with the proper keyboard & screen, this Android device is a full-blown Linux development box.
