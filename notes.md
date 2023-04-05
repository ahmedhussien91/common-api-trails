# target

- we need to download and try the **common-api stack** with some application software

# progress

- install Requirements 

  ```sh
  sudo apt-get install libboost-all-dev
  sudo apt-get install asciidoc source-highlight doxygen graphviz cmake cmake-qt-gui libexpat-dev expat default-jre
  ```

- Install dlt application library

  ```sh
  
  ```

  

we first start with downloading and install the required libraries, we follow 

- https://github.com/COVESA/capicxx-dbus-tools/wiki/CommonAPI-C---D-Bus-in-10-minutes

- https://github.com/COVESA/capicxx-someip-tools/wiki/CommonAPI-C---SomeIP-in-10-minutes

  

## script to install all the dependencies

go to  `$commonapi_path` then `mkdir work` this directory will include all the downloads and the dependencies then execute this script

```sh
#!/bin/sh
export commonapi_path=/opt/commonapi/
sudo apt-get install libboost-all-dev -y
sudo apt-get install asciidoc source-highlight doxygen graphviz -y

# vsomeip
echo "downloading, building and installing vsomeip ...\n"
git clone https://github.com/COVESA/vsomeip.git
cd vsomeip
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX:PATH=$commonapi_path/install -DENABLE_SIGNAL_HANDLING=1 .. 
make 
make install

# capi-core
echo "downloading, building and installing capi-core ...\n"
cd ../..
git clone https://github.com/COVESA/capicxx-core-runtime.git
cd capicxx-core-runtime/
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX:PATH=$commonapi_path/install ..
make
make install

# capi-core-dbus
echo "downloading, building and installing capi-core-dbus ...\n"
cd ../..
git clone https://github.com/GENIVI/capicxx-dbus-runtime.git
# download and install dbus
if [ ! -f dbus-1.10.10.tar.gz ]
then 
	echo "dbus-1.10.10.tar.gz not found, downloading ..."
	wget http://dbus.freedesktop.org/releases/dbus/dbus-1.10.10.tar.gz
else
	echo "dbus-1.10.10.tar.gz found"
fi
if [ ! -d dbus-1.10.10/ ]
then 
	echo "dbus-1.10.10/ not found, executing tar ..."
	tar -xzf dbus-1.10.10.tar.gz
else
	echo "dbus-1.10.10/ found"
fi
cd dbus-1.10.10/
for i in ../capicxx-dbus-runtime/src/dbus-patches/*.patch; do patch -p1 < $i; done
./configure
make
cd ../capicxx-dbus-runtime
mkdir build
cd build
export PKG_CONFIG_PATH="$commonapi_path/dbus-1.10.10" # please set this path
cmake -DUSE_INSTALLED_COMMONAPI=OFF -DUSE_INSTALLED_DBUS=OFF -DCMAKE_INSTALL_PREFIX:PATH=$commonapi_path/install ..
make
make install

# capi-core-someip
echo "downloading, building and installing capi-core-someip ...\n"
cd ../..
git clone https://github.com/GENIVI/capicxx-someip-runtime.git
cd capicxx-someip-runtime
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX:PATH=$commonapi_path/install ..
make 
make install 

# code generator tools (Fails - need time to see why it fails - download releases instead)
# cd ../..
# git clone https://github.com/COVESA/capicxx-core-tools.git
# cd capicxx-core-tools
# cd org.genivi.commonapi.core.releng
# mvn -Dtarget.id=org.genivi.commonapi.core.target clean verify
# cd ../..
# git clone https://github.com/COVESA/capicxx-dbus-tools.git
# cd capicxx-dbus-tools
# cd org.genivi.commonapi.dbus.releng
# mvn -DCOREPATH=../../capicxx-core-tools/org.genivi.commonapi.core.target -Dtarget.id=org.genivi.commonapi.dbus.target clean verify
# cd ../..
# git clone https://github.com/COVESA/capicxx-someip-tools.git
# cd capicxx-someip-tools
# cd org.genivi.commonapi.someip.releng
# mvn -DCOREPATH=../../capicxx-core-tools/org.genivi.commonapi.core.target -Dtarget.id=org.genivi.commonapi.someip.target clean verify
# cd ../..

cd $commonapi_path
mkdir generators
cd generators
echo "downloading generators ...\n"
if [ ! -f commonapi_core_generator.zip ]
then 
	echo "commonapi_core_generator.zip not found, downloading ..."
	wget https://github.com/COVESA/capicxx-core-tools/releases/download/3.2.0.1/commonapi_core_generator.zip
else
	echo "commonapi_core_generator.zip found"
fi

if [ ! -f commonapi_someip_generator.zip ]
then 
	echo "commonapi_someip_generator.zip not found, downloading ..."
	wget https://github.com/COVESA/capicxx-someip-tools/releases/download/3.2.0.1/commonapi_someip_generator.zip
else
	echo "commonapi_someip_generator.zip found"
fi

if [ ! -f commonapi_dbus_generator.zip ]
then 
	echo "commonapi_dbus_generator.zip not found, downloading ..."
	wget https://github.com/COVESA/capicxx-dbus-tools/releases/download/3.2.0/commonapi_dbus_generator.zip
else
	echo "commonapi_dbus_generator.zip found"
fi

if [ ! -d commonapi_core_gen/ ]
then 
	echo "commonapi_core_gen/ not found, executing tar ..."
	mkdir commonapi_core_gen
	unzip commonapi_core_generator.zip -d commonapi_core_gen
else
	echo "commonapi_core_gen/ found"
fi

if [ ! -d commonapi_someip_gen/ ]
then 
	echo "commonapi_someip_gen/ not found, executing tar ..."
	mkdir commonapi_someip_gen
	unzip commonapi_someip_generator.zip -d commonapi_someip_gen
else
	echo "commonapi_someip_gen/ found"
fi

if [ ! -d commonapi_dbus_gen/ ]
then 
	echo "commonapi_dbus_gen/ not found, executing tar ..."
	mkdir commonapi_dbus_gen
	unzip commonapi_dbus_generator.zip -d commonapi_dbus_gen
else
	echo "commonapi_dbus_gen/ found"
fi

# create project folder
cd $commonapi_path
mkdir project 
cd project
mkdir fidl
cd fidl
echo "package commonapi

interface HelloWorld {
  version {major 1 minor 0}
  method sayHello {
    in {
      String name
    }
    out {
      String message
    }
  }
}" > HelloWorld.fidl

# service files 
cd $commonapi_path/project
mkdir server
cd server
echo "// HelloWorldService.cpp
#include <iostream>
#include <thread>
#include <CommonAPI/CommonAPI.hpp>
#include "HelloWorldStubImpl.hpp"

using namespace std;

int main() {
    std::shared_ptr<CommonAPI::Runtime> runtime = CommonAPI::Runtime::get();
    std::shared_ptr<HelloWorldStubImpl> myService =
    	std::make_shared<HelloWorldStubImpl>();
    runtime->registerService(\"local\", \"test\", myService);
    std::cout << \"Successfully Registered Service!\" << std::endl;

    while (true) {
        std::cout << \"Waiting for calls... (Abort with CTRL+C)\" << std::endl;
        std::this_thread::sleep_for(std::chrono::seconds(30));
    }
    return 0;
 }" > HelloWorldService.cpp
 
echo "// HelloWorldStubImpl.hpp
#ifndef HELLOWORLDSTUBIMPL_H_
#define HELLOWORLDSTUBIMPL_H_
#include <CommonAPI/CommonAPI.hpp>
#include <v1/commonapi/HelloWorldStubDefault.hpp>

class HelloWorldStubImpl: public v1_0::commonapi::HelloWorldStubDefault {
public:
    HelloWorldStubImpl();
    virtual ~HelloWorldStubImpl();
    virtual void sayHello(const std::shared_ptr<CommonAPI::ClientId> _client,
    	std::string _name, sayHelloReply_t _return);
};
#endif /* HELLOWORLDSTUBIMPL_H_ */" > HelloWorldStubImpl.hpp

echo "// HelloWorldStubImpl.cpp
#include "HelloWorldStubImpl.hpp"

HelloWorldStubImpl::HelloWorldStubImpl() { }
HelloWorldStubImpl::~HelloWorldStubImpl() { }

void HelloWorldStubImpl::sayHello(const std::shared_ptr<CommonAPI::ClientId> _client,
	std::string _name, sayHelloReply_t _reply) {
	    std::stringstream messageStream;
	    messageStream << \"Hello \" << _name << \"!\";
	    std::cout << \"sayHello('\" << _name << \"'): '\" << messageStream.str() << \"'\n\";

    _reply(messageStream.str());
};" > HelloWorldStubImpl.cpp

# client file
cd $commonapi_path/project
mkdir client
cd client
echo "// HelloWorldClient.cpp
#include <iostream>
#include <string>
#include <unistd.h>
#include <CommonAPI/CommonAPI.hpp>
#include <v1/commonapi/HelloWorldProxy.hpp>

using namespace v1_0::commonapi;

int main() {
    std::shared_ptr < CommonAPI::Runtime > runtime = CommonAPI::Runtime::get();
    std::shared_ptr<HelloWorldProxy<>> myProxy =
    	runtime->buildProxy<HelloWorldProxy>(\"local\", \"test\");

    std::cout << \"Checking availability!\" << std::endl;
    while (!myProxy->isAvailable())
        usleep(10);
    std::cout << \"Available...\" << std::endl;

    CommonAPI::CallStatus callStatus;
    std::string returnMessage;
    myProxy->sayHello(\"Bob\", callStatus, returnMessage);
    std::cout << \"Got message: '\" << returnMessage << \"'\n\";
    return 0;
}" > HelloWorldClient.cpp

# generate files 
cd $commonapi_path/project
../generators/commonapi_core_gen/commonapi-core-generator-linux-x86 --skel --dest-common ./src-gen/core/common --dest-proxy ./src-gen/core/proxy --dest-stub ./src-gen/core/stub --dest-skel ./src-gen/core/skel ./fidl/HelloWorld.fidl

../generators/commonapi_dbus_gen/commonapi-dbus-generator-linux-x86 --dest-common ./src-gen/dbus/common --dest-proxy ./src-gen/dbus/proxy --dest-stub ./src-gen/dbus/stub  ./fidl/HelloWorld.fidl

```



## project creation

https://github.com/COVESA/capicxx-dbus-tools/wiki/CommonAPI-C---D-Bus-in-10-minutes

### fidl

```sh
# from $commonapi_path
mkdir project 
cd project
mkdir fidl
cd fidl
vim HelloWorld.fidl
```

write in file - common file between server and client

```java
package commonapi

interface HelloWorld {
  version {major 1 minor 0}
  method sayHello {
    in {
      String name
    }
    out {
      String message
    }
  }
}
```

### generation

```sh
cd $commonapi_path/project
../generators/commonapi_core_gen/commonapi-core-generator-linux-x86 --skel --dest-common ./src-gen/core/common --dest-proxy ./src-gen/core/proxy --dest-stub ./src-gen/core/stub --dest-skel ./src-gen/core/skel ./fidl/HelloWorld.fidl

../generators/commonapi_dbus_gen/commonapi-dbus-generator-linux-x86 --dest-common ./src-gen/dbus/common --dest-proxy ./src-gen/dbus/proxy --dest-stub ./src-gen/dbus/stub  ./fidl/HelloWorld.fidl

../generators/commonapi_someip_gen/commonapi-someip-generator-linux-x86 --dest-common ./src-gen/someip/common --dest-proxy ./src-gen/someip/proxy --dest-stub ./src-gen/someip/stub ./fidl/HelloWorld.fdepl
```



### client

```sh
# from ``$commonapi_path/project`
mkdir client
vim HelloWorldClient.cpp # write the file 
vim CMakeLists.txt # write the file 
```

### server

```sh
# from ``$commonapi_path/project`
mkdir server
vim HelloWorldService.cpp # write the file 
vim HelloWorldStubImpl.hpp # write the file 
vim HelloWorldStubImpl.cpp # write the file 
vim CMakeLists.txt # write the file 
```



# links 

https://github.com/COVESA/capicxx-core-tools/wiki

https://github.com/COVESA/capicxx-dbus-tools/wiki/CommonAPI-C---D-Bus-in-10-minutes

https://github.com/COVESA/capicxx-someip-tools/wiki/CommonAPI-C---SomeIP-in-10-minutes

https://github.com/COVESA/capicxx-core-tools/wiki/Loading-Bindings-And-Libraries

https://github.com/COVESA/capicxx-core-tools/wiki/Addresstranslation

https://github.com/COVESA/vsomeip/wiki/vsomeip-in-10-minutes

[COMMON API CPP User Guide](https://usermanual.wiki/Document/CommonAPICppUserGuide.113855339.pdf)

[COMMON API DBUS GUIDE](https://usermanual.wiki/Document/CommonAPIDBusCppUserGuide.1823987351.pdf)

[dbus tutorial](https://dbus.freedesktop.org/doc/dbus-tutorial.html)

[franca github](https://github.com/franca/franca)

[franca user guide](https://drive.google.com/drive/folders/0B7JseVbR6jvhUnhLOUM5ZGxOOG8?resourcekey=0-U-X53hicOvlqAZCG86dCUQ)
