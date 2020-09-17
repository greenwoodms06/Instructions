# FMI-Library Installation Tutorial for Windows

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

## libzip
1. Download and extract fmi-library directly from browser or open WSL terminal and enter:
    ```
    git https://github.com/modelon-community/fmi-library.git
    ```
    - Note if downloading/extracting make sure to change path to files is `/temp/fmi-library/` (i.e., no nested directories or other weird names which come from downloading zip files)
  
# Install Libraries

## fmi-library
1. Right click on the fmi-library folder and open with visual studio
2. Under `Build` select the installation option

# Simulate using FMI-Library

1. Grab the `test.fmu` and place in `C:/PATH/temp/resources`.
   - Note that FMU was compiled with VS2019 compiler and may not work elsewhere. Feel free to use any FMU you are able to generate.
2. Open Visual Studio 2019 and start a new console project and give it a name. We'll use `Demo-FMI-Library`
3. Right click the project and select `Properties` and make the following modifications.
   - Under `C/C++ > General > Additional Include Directories` set to 
        ```
        C:\PATH\temp\fmi-library\out\build\install\include
        ```
   - Under `Linker > General > Additional Library Dependencies` set to 
        ```
        C:\PATH\temp\out\build\install\lib
        ```
   - Under `Linker > Input > Additional Dependencies` set to 
        ```
        fmilib.lib
		fmilib_shared.lib
        ```
   - Under `Build Event > Post-Build Event > Command Line` set to 
        ```
        xcopy /y /d "C:\PATH\temp\out\build\install\lib\fmilib_shared.dll" "$(OutDir)"
        ```

4. Under construction... current working code not available :(
