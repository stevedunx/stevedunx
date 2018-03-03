# Getting Pritunl installed on Raspbian stretch

## Pritunl

I wanted to install Pritunl on my Raspberry Pi. The Pritunl repos do not support the Raspberry Pi architecture, so it needs to be compiled from source. Below is the command that I ran (in a tidied form) on Raspbian in order to get Pritunl installed on my Pi.

Adapted from https://github.com/pritunl/pritunl

My `pip --version` is `pip 9.0.1 from /usr/lib/python2.7/dist-packages (python 2.7)`, so I've used python 2.7 in the commands below.

Note that pritunl requires Mongo DB 2.6+, but the latest in Raspbian repos is 2.4.

```
export VERSION=X.XX.XX.XX # Set current pritunl version here

sudo apt-get install libffi-dev libssl-dev
sudo apt-get install golang git bzr python-dev python-pip net-tools openvpn bridge-utils psmisc

echo "export GOPATH=/go" >> ~/.bash_profile
source ~/.bash_profile
# sudo -E will use the local users GOPATH
sudo -E go get -u -v github.com/pritunl/pritunl-dns
sudo -E go get -u -v github.com/pritunl/pritunl-web
sudo ln -s /go/bin/pritunl-dns /usr/local/bin/pritunl-dns
sudo ln -s /go/bin/pritunl-web /usr/local/bin/pritunl-web

wget https://github.com/pritunl/pritunl/archive/$VERSION.tar.gz
tar xf $VERSION.tar.gz
cd pritunl-$VERSION
python2.7 setup.py build
sudo pip install -r requirements.txt # if you run this without sudo, it will pip install for the local user
sudo pip install --upgrade google-auth-oauthlib
sudo python2.7 setup.py install

sudo systemctl daemon-reload
sudo systemctl start pritunl
sudo systemctl enable pritunl
```

You can check pritunl runs with `sudo pritunl start`. For the service logs, use `sudo pritunl logs` or `sudo service pritunl status`.

When I ran `sudo service pritunl status`, I found `pritunl.service: Failed at step EXEC spawning /usr/bin/pritunl: No such file or directory`. So, I needed to `sudo nano /etc/systemd/system/pritunl.service` and edit it so that `ExecStart=/usr/local/bin/pritunl start`. Then:

```
sudo systemctl daemon-reload
sudo systemctl start pritunl
```

## Mongo DB

The last hurdle now is to get an up-to-date MongoDB on the Pi. Note that pritunl requires Mongo DB 2.6+, but the latest in Raspbian repos is 2.4. 

I shall probably use Docker 

```
# Get docker
curl -sSL https://get.docker.com | sh
# Get a precompiled mongo image into docker
sudo docker pull dhermanns/rpi-mongo
```

# Sources:
* `sudo -E` https://stackoverflow.com/questions/19854835/gopath-environment-variable-not-set#comment76476872_19864442
* Install `libffi-dev` https://stackoverflow.com/a/31508663
* Install `libssl-dev` https://github.com/certbot/certbot/issues/37#issuecomment-64003137
* `upgrade google-auth-oauthlib` because `from pyasn1.type import opentype` fails https://stackoverflow.com/a/47602779
* `go get -u -v` because it looks like it's doing nothing https://github.com/golang/go/issues/17959
* Options for MongoDB on the Pi https://raspberrypi.stackexchange.com/a/67996
* Get docker https://blog.alexellis.io/getting-started-with-docker-on-raspberry-pi/
* MongoDB RPi Docker image https://github.com/dhermanns/rpi-mongo
