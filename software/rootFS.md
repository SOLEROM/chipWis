# dev env build

## build container

```

```

```
## for more easy use of python linked to 3
ln -s /usr/bin/python3 /usr/bin/python
```


## add deps

```

sudo apt install python3-pip
sudo apt install libaec-dev



sudo apt install libusb-dev make
sudo apt install avr-libc gcc-avr
sudo apt install gcc-arm-none-eabi

fix for boinic
==============
cd /chipWS/deps_libnewlib
wget https://mirrors.kernel.org/ubuntu/pool/universe/n/newlib/libnewlib-dev_3.3.0-0ubuntu1_all.deb    
wget https://mirrors.kernel.org/ubuntu/pool/universe/n/newlib/libnewlib-arm-none-eabi_3.3.0-0ubuntu1_all.deb   
sudo dpkg -i libnewlib-arm-none-eabi_3.3.0-0ubuntu1_all.deb  libnewlib-dev_3.3.0-0ubuntu1_all.deb 


```

## cw src

```
cd /chipWS
## use --recursive to get the submodules
git clone --recursive https://github.com/newaetech/chipwhisperer.git cwGIT
```


## udev rules

```
cp /chipWS/cwGIT/hardware/99-newae.rules /etc/udev/rules.d/	 ??? container or host ???

sudo usermod -a -G plugdev YOUR-USERNAME
sudo udevadm control --reload-rules
log out & in again

```

## cw code

get code deps
```
cd /chipWS/cwGIT
python -m pip install -e . --user
```

install jupyter

```
cd /chipWS/cwGIT
pip3 install -r jupyter/requirements.txt --user
```




