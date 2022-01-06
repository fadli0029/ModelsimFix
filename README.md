## The issue: 
The free version of __ModelSim-Intel FPGA Edition__ that came with __Quartus Prime Lite 19.1__ only works with 32-bit system for Linux. The Quartus Prime itself works perfect.

---

## Solution
This solution assumes you have downloaded Quartus Prime Lite Edition via "Combined Files" [here](https://fpgasoftware.intel.com/19.1/?edition=lite&platform=linux)  

And if it matters, I'm on Ubuntu 20.04.

First, install some dependencies:
```bash
sudo dpkg --add-architecture i386
sudo apt-get update
sudo apt-get install build-essential
sudo apt-get install gcc-multilib g++-multilib lib32z1 lib32stdc++6 lib32gcc1 expat:i386 fontconfig:i386 libfreetype6:i386 libexpat1:i386 libc6:i386 libgtk-3-0:i386 libcanberra0:i386 libpng16-16:i386 libice6:i386 libsm6:i386 libncurses5:i386 zlib1g:i386 libx11-6:i386 libxau6:i386 libxdmcp6:i386 libxext6:i386 libxft2:i386 libxrender1:i386 libxt6:i386 libxtst6:i386
```

Next, download `freetype` [here](https://sourceforge.net/projects/freetype/files/freetype2/2.4.12/freetype-2.4.12.tar.bz2/download)  

Extract & build it:
``` bash
tar -xjvf freetype-2.4.12.tar.bz2
cd freetype-2.4.12
./configure --build=i686-pc-linux-gnu "CFLAGS=-m32" "CXXFLAGS=-m32" "LDFLAGS=-m32"
make -j$(nproc)
```

Navigate to your `modelsim_ase` directory. I will call it `YOUR_MODELSIM_ASE_DIR` hereafter.  
Then inside `YOUR_MODELSIM_ASE_DIR` do the following:
```bash
sudo mkdir lib32
sudo cp ~/freetype-2.4.12/objs/.libs/libfreetype.so* ./lib32
```

Modify the `vsim` script in `YOUR_MODELSIM_ASE_DIR/bin/`.  

Find the following line:
```
dir=`dirname "$arg0"`
```
Then add the following line below it:
```
export LD_LIBRARY_PATH=${dir}/lib32
```

Next, find the following line:
```
vco="linux_rh60"
```
And change to only says "linux":
```
vco="linux"
```

Lastly, find the following line:
```
mode=${MTI_VCO_MODE:-" "}
```
And replace it with the following line:
```
mode=${MTI_VCO_MODE:-"32"}
```

Try executing modelsim by running the following line in `YOUR_MODELSIM_ASE_DIR/bin/`:
```
./vsim
```

__If you get some kind of `libfontconfig` error, then you still have some work to do, hang on there. This is the last step.__  

Download [libfontconfig1_2.12.6-0ubuntu2_i386.deb](http://ubuntu.mirrors.tds.net/ubuntu/pool/main/f/fontconfig/libfontconfig1_2.12.6-0ubuntu2_i386.deb) and extract it (if it's downloaded in your `Downloads` folder, then run the following command in there):
``` bash
mkdir libfontconfigextracted
dpkg -x libfontconfig1_2.12.6-0ubuntu2_i386.deb libfontconfigextracted
```

When we installed the dependencies earlier, we should have `i386-linux-gnu` directory. In my case, it's here: `/usr/lib/i386-linux-gnu/`. Navigate to that directory.

Then rename these two files, `libfontconfig.so.1` and `libfontconfig.so.1.12.0`, because we need to replace these later. Run the following command:
```bash
mv libfontconfig.so.1 libfontconfig.so.1_old
mv libfontconfig.so.1.12.0 libfontconfig.so.1.12.0_old
```

Finally go back to our recently downloaded `libfontconfig`, wherever you put it.  
In there, we'll need `libfontconfig.so.1` and `libfontconfig.so.1.10.1` to replace the ones we renamed earlier. To do this, go to the folder where you extracted the `libfontconfig1_2.12.6-0ubuntu2_i386.deb` we downloaded earlier. If you've been following all the guidelines here, it should be in your `Downloads` folder, inside `libfontconfigextracted`. Then, inside `libfontconfigextracted` run the following command:
```bash
cd usr/lib/i386-linux-gnu/
cp libfontconfig.so.1 /usr/lib/i386-linux-gnu/
cp libfontconfig.so.1.10.1 /usr/lib/i386-linux-gnu/
```

Let's test if it works. Go to `YOUR_MODELSIM_ASE_DIR` and run the following:  
```bash
cd bin
./vsim
```

You will see a bunch of `fontconfig` errors. I am aware of that, but notice that modelsim launches perfectly (If you know how to avoid this fontconfig errors, let me know!). And don't worry because when you launched it from the Quartus software via "run simulation", you shouldn't see this error.  


### Citations:  

[https://pcotret.github.io/modelsim-ubuntu/](https://pcotret.github.io/modelsim-ubuntu/)  

[http://twoerner.blogspot.com/2017/10/running-modelsim-altera-from-quartus.html](http://twoerner.blogspot.com/2017/10/running-modelsim-altera-from-quartus.html)
