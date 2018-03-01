# Getting Pritunl installed on Raspbian stretch

I wanted to install Pritunl on my Raspberry Pi. The Pritunl repos do not support the Raspberry Pi architecture, so it needs to be compiled from source. Below is the command that I ran (in a tidied form) on Raspbian in order to get Pritunl installed on my Pi.

Adapted from https://github.com/pritunl/pritunl
```
export VERSION=X.XX.XX.XX # Set current pritunl version here

# see https://github.com/certbot/certbot/issues/37#issuecomment-64003137
# see https://stackoverflow.com/a/31508663
sudo apt-get install libffi-dev libssl-dev mongodb-server
sudo apt-get install golang git bzr python-dev python-pip net-tools openvpn bridge-utils psmisc

echo "export GOPATH=/go" >> ~/.bash_profile
source ~/.bash_profile
go get github.com/pritunl/pritunl-dns
go get github.com/pritunl/pritunl-web
sudo ln -s /go/bin/pritunl-dns /usr/local/bin/pritunl-dns
sudo ln -s /go/bin/pritunl-web /usr/local/bin/pritunl-web

wget https://github.com/pritunl/pritunl/archive/$VERSION.tar.gz
tar xf $VERSION.tar.gz
cd pritunl-$VERSION
python2 setup.py build
pip install -r requirements.txt
sudo python2 setup.py install

sudo systemctl daemon-reload
sudo systemctl start mongodb pritunl
sudo systemctl enable mongodb pritunl

```
