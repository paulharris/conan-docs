.. _workflows:

Workflows
=========

.. caution::

    We are actively working to finalize the *Conan 2.0 Release*. Some of the information on this page references
    **deprecated** features which will not be carried forward with the new release. It's important to check the 
    :ref:`Migration Guidelines<conan2_migration_guide>` to ensure you are using the most up to date features.

This section summarizes some possible layouts and workflows when using Conan together with other
tools as an end-user for installing and consuming existing packages. To create your own
packages, please refer to :ref:`Creating Packages <packaging>`.

Whether you are working on a single configuration or a multi configuration project, in both cases, the recommended approach is to have a conanfile (either .py or .txt) at the root of
your project.

Single configuration
--------------------

When working with a  single configuration, your conanfile will be quite simple as shown in the examples and tutorials we have used so far in this user guide. For example, in
:ref:`Getting started<getting_started>`, we showed how you can run the :command:`conan install ..` command inside the *build* folder resulting in the *conaninfo.txt* and
*conanbuildinfo.cmake* files being generated there too. Note that the build folder is temporary, so you should exclude it from version control to exclude these temporary files.

**Out-of-source builds** are also supported. Let's look at a simple example:

.. code-block:: bash

    $ git clone https://github.com/conan-io/examples.git
    $ cd libraries/poco
    $ conan install ./md5 --install-folder=md5_build

This will result in the following layout:

.. code-block:: text

    md5_build
        conaninfo.txt
        conanbuildinfo.txt
        conanbuildinfo.cmake
    md5
        CMakeLists.txt  # If using cmake, but can be Makefile, sln...
        README.md
        conanfile.txt
        md5.cpp

Now you are ready to build:

.. code-block:: bash

    $ cd md5_build
    $ cmake ../md5 -G "Visual Studio 15 Win64"  # or other generator
    $ cmake --build . --config Release
    $ ./bin/md5
    > c3fcd3d76192e4007dfb496cca67e13b

We have created a separate build configuration of the project without affecting the original
source directory in any way. The benefit is that we can freely experiment with the configuration: 
We can clear the build folder and build another. For example, changing the build type to Debug:

.. code-block:: bash

    $ rm -rf *
    $ conan install ../md5 -s build_type=Debug
    $ cmake ../md5 -G "Visual Studio 15 Win64"
    $ cmake --build . --config Debug
    $ ./bin/md5
    > c3fcd3d76192e4007dfb496cca67e13b

Multi configuration
-------------------

You can also manage different configurations, whether in-source or out of source, and switch between
them without having to re-issue the :command:`conan install` command (Note however, that even if you did have to run :command:`conan install` again, since subsequent runs use the same parameters, they would be very fast since packages would already have been installed in the local cache rather than in the project)

.. code-block:: bash

    $ git clone git@github.com:conan-io/examples
    $ cd libraries/poco
    $ conan install md5 -s build_type=Debug -if md5_build_debug
    $ conan install md5 -s build_type=Release -if md5_build_release

    $ cd md5_build_debug && cmake ../md5 -G "Visual Studio 15 Win64" && cd ../..
    $ cd md5_build_release && cmake ../md5 -G "Visual Studio 15 Win64" && cd ../..

.. note::

    You can either use the ``--install-folder`` or ``-if`` flags to specify where to generate the output files, or
    manually create the output directory and navigate to it before executing the :command:`conan install` command.

So the layout will be:

.. code-block:: text

    md5_build_debug
        conaninfo.txt
        conanbuildinfo.txt
        conanbuildinfo.cmake
        CMakeCache.txt # and other cmake files
    md5_build_release
        conaninfo.txt
        conanbuildinfo.txt
        conanbuildinfo.cmake
        CMakeCache.txt # and other cmake files
    example-poco-timer
        CMakeLists.txt  # If using cmake, but can be Makefile, sln...
        README.md
        conanfile.txt
        md5.cpp

Now you can switch between your build configurations in exactly the same way you do for CMake or
other build systems, by moving to the folder in which the build configuration is located, because the Conan
configuration files for that build configuration will also be there.

.. code-block:: bash

    $ cd md5_build_debug && cmake --build . --config Debug && cd ../..
    $ cd md5_build_release && cmake --build . --config Release && cd ../..

Note that the CMake ``include()`` of your project must be prefixed with the current cmake binary
directory, otherwise it will not find the necessary file:

.. code-block:: cmake

    include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
    conan_basic_setup()

.. seealso::

    There are two generators, ``cmake_multi`` and ``visual_studio_multi`` that could help to avoid the
    context switch and using Debug and Release configurations simultaneously. Read more about them in
    :ref:`cmakemulti_generator` and :ref:`visual_studio_multi` 
