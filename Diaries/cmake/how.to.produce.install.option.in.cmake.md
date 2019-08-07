There are two ways to use this variable:

    passing it as a command line argument just like Job mentioned:
```
    cmake -DCMAKE_INSTALL_PREFIX=< install_path > ..
```

    assigning value to it in CMakeLists.txt:
```
    SET(CMAKE_INSTALL_PREFIX < install_path >)
```
    But do remember to place it BEFORE PROJECT(< project_name>) command, otherwise it will not work!


