## KLAYOUT intallation

go https://www.klayout.de/build.html download the pkg deb and install
```
sudo dpkg -i klayout_0.29.12-1_amd64.deb
```
if there are dependencies problems, update the system operate and rerun the command. \
or install manually dependencies
```
 sudo apt-get -f install
```
and re run 
```
sudo dpkg -i klayout_0.29.12-1_amd64.deb
```

### OPENROAD
I was using ubuntu 24.04 and I had issues trying to install OpenRoad
```
https://github.com/The-OpenROAD-Project/OpenROAD/tree/master
```

I tried to install prebuild binaries but was no available for ubuntu v24.04

Then I followed the "Build Locally" from
```
https://github.com/The-OpenROAD-Project/OpenROAD/blob/master/docs/user/Build.md
```
I faced problems that I described here, and show how to solve it
```
https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/issues/2904
```

The I ran:
```
sudo ./etc/DependencyInstaller.sh -all
sudo ./etc/Build.sh
```

### OpenROAD-flow-scripts
I saw that for start to know the use of OpenRoad I need to see same examples of how to use it. for that reason I proceed to install OpenROAD-flow-scripts from
```
https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/tree/master
```
