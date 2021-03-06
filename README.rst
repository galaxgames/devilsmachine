
devilsmachine
=============

devilsmachine is a content preprocessor framework that makes integrating preprocessors easy. devilsmachine has first
class support for CMake and eliminates the need to write burdensome boilerplate CMake code for each proceprocesor you
use. Preprocessors are configured through a deadly simple config file that devilsmachine uses to do the heavy CMake
lifting.

devilsmachine makes writing preprocessors easy too. Preprocessors are written as python modules containing a
preprocessor class. The devilsmachine CMake module handles all of the work for configuring targets for each file a
preprocessor touches. It's also incredibly easy to use any pip python module, as the CMake module handles setting up a
virtual environment and ensuring the required pip packages are installed. All you need to do is define your pip
dependencies in the devilsmachine config file, in a subsection which is essentially just an embedded requirements.txt
file.

Integrating devilsmachine with your CMake Project
=================================================

First, you must install devilsmachine using devilsmachine's CMakeLists.txt file.

.. code:: sh

    git clone git@github.com:galaxgames/devilsmachine.git
    cd devilsmachine
    mkdir build
    cd build
    cmake -G "Unix Makefiles" ..
    cmake --build . --target install

This will install the devilsmachine python library and CMake to your prefix. By default, this prefix is `/usr/local/`.
It is recommended that you use a different prefix path for each project. You can change this prefix by passing
`-DCMAKE_INSTALL_PREFIX=/your/project/prefix/path` into the first `cmake -G` command. I also recomment you check out
dew_ for an example of how to manage CMake dependencies (or you could just use dew if you're brave enough :wink:)

.. _dew: https://github.com/sfuller/dew

Next, Insert the following into your project's CMakeLists.txt file:

.. code:: cmake

    include(devilsmachine)

    # Assemble a list of all the source files devilsmachine should evaluate and produce output for.
    # For example, you might have a have a directory (in this example named 'content') that contains all of the files
    # you need processed.
    file(GLOB_RECURSE MY_PROJECT_CONTENT_SOURCE_FILES "${CMAKE_CURRENT_SOURCE_DIR}/content/*")

    # Next, use this function to have devilsmachine work it's magic!
    add_devilsmachine_commands(
        "${MY_PROJECT_CONTENT_SOURCE_FILES}"
        MY_PROJECT_CONTENT_GENERATED_FILES      # This variable will be set with the filenames of all the files
                                                # that will be generated by devils machine at build time.
        "${CMAKE_CURRENT_SOURCE_DIR}/dmconfig"  # See below for more info about this file
        "${CMAKE_CURRENT_SOURCE_DIR}/content"   # A path to directory which contains all of of your content files.
                                                # Used to help structure the generated output directory below
        "${CMAKE_CURRENT_BINARY_DIR}/content"   # A path to a directory where all of the generated artifacts should go.
    )

    # Now create a target which depends on all of the generated files given to us from the above function.
    # NOTE: In the future, the function above might be able to do this for you. Stay tuned :)
    add_custom_target(
        my_project_content
        DEPENDS ${MY_PROJECT_CONTENT_GENERATED_FILES}
        SOURCES ${MY_PROJECT_CONTENT_SOURCE_FILES}
    )

    # Gotta make sure our main target depends on the target we created above using devilsmachine.
    # Otherwise the generated files your main target needs won't be generated!
    add_dependencies(my_project my_project_content)

And finally, we need to define a config file for devilsmachine, to specify what it should preprocess and how it should
preprocess.

.. code::

    #
    # Config file for devilsmachine content preprocessor
    # https://github.com/galaxgames/devilsmachine
    #

    [dependencies]
    pillow==5.2.0

    [processors]
    tga:	devilsmachine.stockmodules:Copy
    glsl:	devilsmachine.stockmodules.glsl:CompileGLSL
    level:  my_project_preprocessor_module:LevelPreprocessor
    png:    my_project_preprocessor_module:PngPreprocessor

In devilsmachine, preprocessor selection is based on the source file file extension. Note that there may be more
complex preprocessor determination features in the future. In this example, we dictate that all .tga files will simply
be copied to the output directory using the built-in Copy module. We then dictacte that all .glsl files will be
processed by the built-in glsl module. This glsl module compiles the source file, and outputs the compiled .spirv file
in the output directory. Note that devilsmachine does not actually contain a glsl compiler, but requires an external
tool. The devilsmachine CMake module will look for automaticly, as preprocessor modules can list requirement tools to be
found by CMake's find_program() machinery.

Next, we have some custom modules. These modules are defined by python modules which live in your project. devilsmachine
includes your project root in the PYTHONPATH, allowing you to specify any python modules you have living in your
project.

The syntax for a preprocessor configuration is:

`file_extension: module_name:preprocessor_class_name`

The dependencies section of the config file defines a requirements.txt file which will be given to pip when installing
all needed dependency libraries to the project's virtualenv. In our example, our project's PngPreprocessor uses pillow
as a dependency to do convert png images into another format. Note: In the future, dependencies may be defined in the
modules, requiring less configuration for the user.

And that's about it!
====================
Thanks for reading my readme! If this project gets more popular I will put together more thorough documentation of
course. But until then, have an *amazing* day!