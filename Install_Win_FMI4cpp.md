# FMI4cpp Installation Tutorial for Windows

This tutorial has been tested only with Visual Studio 2019 on Windows 10.

## Additional Resources
- https://docs.microsoft.com/en-us/cpp/build/walkthrough-creating-and-using-a-dynamic-link-library-cpp?view=vs-2019

# Prepare Directory
1. Create the directory to store libraries. Well use `temp` for this example.
```
cd C:/PATH/
mkdir temp
cd temp
```
- `PATH` is the location you wish to create the `temp` folder

# Download Libraries

## BOOST
1. Download BOOST directly from browser or open WSL terminal and enter:
    ```
    wget https://sourceforge.net/projects/boost/files/boost-binaries/1.74.0/boost_1_74_0-msvc-14.1-64.exe
    ```

## zlib
1. Download zlib directly from browser or open WSL terminal and enter:
    ```
    cd c:/PATH/
    wget https://zlib.net/zlib1211.zip
    unzip zlib1211 -d zlib
    ```

## libzip
1. Download and extract libzip directly from browser or open WSL terminal and enter:
    ```
    git clone https://github.com/nih-at/libzip.git
    ```
    - Note if downloading/extracting make sure to change path to files is `/temp/libzip/` (i.e., no nested directories or other weird names which come from downloading zip files)
  
## FMI4cpp
1. Download and extract FMI4cpp directly from browser or open WSL terminal and enter:
    ```
    git clone https://github.com/NTNU-IHB/FMI4cpp.git
    ```
    - Note if downloading/extracting make sure to change path to files is `/temp/FMI4cpp/` (i.e., no nested directories or other weird names which come from downloading zip files)
  

# Install Libraries

## BOOST
1. Run the executable and then click Finish. Default installation location works (ie., `c:/local/boost_VERSION`) 
2. Open CMD terminal and enter:
    ```
    cd C:/local/boost_1_74_0
    .\bootstrap.bat
    .\b2
    ```

## Set temporary environment variables

1. Open `x64 Native Tools Command Prompt for VS 2019` and copy-paste to set temporary environment variables:
    ```
    set PARENTPATH=C:/PATH/temp
    cd %PARENTPATH%
    set ZLIBPATH=%PARENTPATH%/zlib/zlib_1_2_11
    set LIBZIPPATH=%PARENTPATH%/libzip
    set BOOSTPATH=C:/local/boost_1_74_0
    set FMI4CPPPath=%PARENTPATH%/FMI4cpp
    ```
    - Make sure to update `PATH` and `temp` and the versions of the other libraries being used (e.g,. `boost_1_71_0`)

## zlib
1. Make sure to set the temporary environment variables indicated above if not already done.
2. Build and Install
    ```
    cd %PARENTPATH%
    cd zlib
    mkdir build
    cd build
    cmake -DCMAKE_INSTALL_PREFIX=%ZLIBPATH% -DCMAKE_GENERATOR_PLATFORM=x64 ..
    cmake --build . --target install
    ```

## libzip
1. Make sure to set the temporary environment variables indicated above if not already done.
2. Build and Install
    ```
    cd %PARENTPATH%
    cd zlib
    mkdir build
    cd build
    cmake -DCMAKE_INSTALL_PREFIX=%LIBZIPPATH% -DZLIB_ROOT=%ZLIBPATH% -DCMAKE_GENERATOR_PLATFORM=x64 ..
    cmake --build . --target install
    ```

## FMI4cpp
1. Make sure to set the temporary environment variables indicated above if not already done.
2. Build and Install
    ```
    cd %PARENTPATH%
    cd zlib
    mkdir build
    cd build
    cmake -DCMAKE_INSTALL_PREFIX=%FMI4CPPPath%/install -DZLIB_ROOT=%ZLIBPATH% -DBOOST_ROOT=%BOOSTPATH% -DLIBZIP_DIR=%LIBZIPPATH% -DCMAKE_GENERATOR_PLATFORM=x64 ..
    cmake --build . --target install
    ```
    - A common error in this stage may be a missing include statement. If  `error C####: 'runtime_error': is not a member of 'std'` occurs during the build/install stage, go to `\FMI4cpp\src\fmi2\xml\model_description.cpp` and add `#include <stdexcept>`

# Simulate using FMI4cpp

1. Grab the `test.fmu` and place in `C:/PATH/temp/resources`.
   - Note that FMU was compiled with VS2019 compiler and may not work elsewhere. Feel free to use any FMU you are able to generate.
2. Open Visual Studio 2019 and start a new console project and give it a name. We'll use `Demo-FMI4cpp`
3. Right click the project and select `Properties` and make the following modifications.
   - Under `C/C++ > General > Additional Include Directories` set to 
        ```
        C:\PATH\temp\FMI4cpp\include
        ```
   - Under `C/C++ > Language > C++ Language Standard` set to 
        ```
        ISO C++17 Standard (/std:c++17)
        ```
   - Under `Linker > General > Additional Library Dependencies` set to 
        ```
        C:\PATH\temp\FMI4cpp\lib
        ```
   - Under `Linker > Input > Additional Dependencies` set to 
        ```
        fmi4cppd.lib
        ```
   - Under `Build Event > Post-Build Event > Command Line` set to 
        ```
        xcopy /y /d "C:\PATH\temp\zlib\zlib_1_2_11\bin\zlibd.dll" "$(OutDir)"
        xcopy /y /d "C:\PATH\temp\libzip\bin\zip.dll" "$(OutDir)"
        xcopy /y /d "C:\PATH\temp\FMI4cpp\bin\fmi4cppd.dll" "$(OutDir)"
        ```

4. If not already created, add a new `.cpp` file and paste the following -(adapted from the FMI4cpp github site):
    ```
    #define NOMINMAX

    #include <fmi4cpp/fmi4cpp.hpp>

    #include <iostream>
    #include <string>

    using namespace fmi4cpp;

    const double stop = 1.0;
    const int intervals = 500;
    const double stepSize = 1.0 / intervals;

    int main()
    {
        const std::string fmuPath = "C:/PATH/temp/resources/test.fmu";

        fmi2::fmu fmu(fmuPath);

        auto cs_fmu = fmu.as_cs_fmu();
        auto md = cs_fmu->get_model_description();

        auto slave1 = cs_fmu->new_instance();
        std::cout << "model_identifier=" << slave1->get_model_description()->model_identifier << std::endl;

        slave1->setup_experiment();
        slave1->enter_initialization_mode();
        slave1->exit_initialization_mode();

        std::vector<fmi2Real> ref(2);
        std::vector<fmi2ValueReference> vr = { md->get_variable_by_name("x").value_reference,
                                            md->get_variable_by_name("y").value_reference };

        double t = 0;
        while ((t = slave1->get_simulation_time()) <= stop) {

            if (!slave1->step(stepSize)) { break; }
            if (!slave1->read_real(vr, ref)) { break; }
            std::cout << "t=" << t << ", x=" << ref[0] << ", y=" << ref[1] << std::endl;
        }

        std::cout << "FMU '" << fmu.model_name() << "' terminated with success: " << (slave1->terminate() == 1 ? "true" : "false") << std::endl;

        return 0;
    }
    ```
    - If there are min/max errors make sure `#define NOMINMAX` is added to the `.cpp` file.
