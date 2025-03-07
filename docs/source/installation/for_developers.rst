For developers
==============

This page is for developers and advanced users. It describes
how to build k2 and run tests.

First, you have to install CMake, CUDA toolkit (with cuDNN), and PyTorch:

  - CMake 3.11.0 and 3.18.0 are known to work. Other CMake versions may work
    but they are not tested.

  - Install PyTorch. PyTorch 1.4.x and above are known to work. Other PyTorch
    versions may work, but they are not tested.

  - Install CUDA toolkit and cuDNN. CUDA 10.1 and above are known to work.
    Other versions are not tested.

  - Your Python version has to be at least 3.6.

Second, let's clone the repository to some path ``/some/path``:

.. code-block:: bash

  cd /some/path
  git clone https://github.com/k2-fsa/k2.git

  # Normally, you would first fork the repo and use
  # git clone https://github.com/your_github_username/k2.git

To build a release version, use:

.. code-block:: bash

  cd /some/path/k2
  mkdir build_release
  cd build_release
  cmake -DCMAKE_BUILD_TYPE=Release ..
  make -j
  export PYTHONPATH=$PWD/../k2/python:$PYTHONPATH # for `import k2`
  export PYTHONPATH=$PWD/lib:$PYTHONPATH # for `import _k2`

  # To test that your build is successful, run
  python3 -c "import k2; print(k2.__file__)"
  # It should print /some/path/k2/k2/python/k2/__init.py

  python3 -c "import torch; import _k2; print(_k2.__file__)"
  # It should print /some/path/k2/build_release/lib/_k2.cpython-38-x86_64-linux-gnu.so
  # (I assume that you're using Python 3.8, so there is a string 38 above)

To build a debug version, use:

.. code-block:: bash

  cd /some/path/k2
  mkdir build_debug
  cd build_debug
  cmake -DCMAKE_BUILD_TYPE=Debug ..
  make -j
  export PYTHONPATH=$PWD/../k2/python:$PYTHONPATH # for `import k2`
  export PYTHONPATH=$PWD/lib:$PYTHONPATH # for `import _k2`

  # To test that your build is successful, run
  python3 -c "import k2; print(k2.__file__)"
  # It should print /some/path/k2/k2/python/k2/__init.py

  python3 -c "import torch; import _k2; print(_k2.__file__)"
  # It should print /some/path/k2/build_debug/lib/_k2.cpython-38-x86_64-linux-gnu.so
  # (I assume that you're using Python 3.8, so there is a string 38 above)

.. HINT::

  You can pass the option ``-DK2_WITH_CUDA=OFF`` to ``cmake`` to build
  a CPU only version of k2.

  It is much faster to build a CPU version than that of building a CUDA
  version. When you are adding new features to k2, we recommend you to
  create a diretory to build a CPU version to test your code. Once it is
  working on CPU, you can create a new directory to build a CUDA version
  to test your code.

  That is, while adding and testing new features, use:

    .. code-block:: bash

      cd k2
      mkdir build-cpu
      cd build-cpu
      cmake -DK2_WITH_CUDA=OFF -DCMAKE_BUILD_TYPE=Debug ..
      make -j5
      export PYTHONPATH=$PWD/../k2/python:$PWD/lib:$PYTHONPATH
      # make test # to test your code

  After it is working for CPU, you can use:

    .. code-block:: bash

      cd k2
      mkdir build-cuda
      cd build-cuda
      cmake -DCMAKE_BUILD_TYPE=Debug ..
      make -j5
      export PYTHONPATH=$PWD/../k2/python:$PWD/lib:$PYTHONPATH
      # make test # to test your code

To run tests, use:

.. code-block:: bash

  cd /some/path/k2/build_release # or switch to build_debug
  make -j
  make test
  # alternatively, you can run
  # ctest -j5

To run a specific C++ test, use:

.. code-block:: bash

  cd /some/path/k2/build_release # or switch to build_debug
  make cu_ragged_test
  # You will find an executable ./bin/cu_ragged_test
  ./cu_ragged_test
  #
  # Use `make help` to find all available C++ tests


  # Inside k2/csrc/ragged_test.cu, there is a test case like below:
  #
  # TEST(RaggedShapeOpsTest, CatMoreAxes) {
  #
  # To run the above test case only, use
  ./cu_ragged_test --gtest_filter="RaggedShapeOpsTest.CatMoreAxes"
  #
  # The option `--gtest_filter` supports regular expressions.
  #
  # Run `./cu_ragged_test --help` to learn more

To run a specific Python test, use:

.. code-block:: bash

  cd /some/path/k2/build_release # or switch to build_debug

  export PYTHONPATH=$PWD/../k2/python:$PYTHONPATH # for `import k2`
  export PYTHONPATH=$PWD/lib:$PYTHONPATH # for `import _k2`

  python3 ../k2/python/tests/index_test.py

  # Alternatively, you can use
  ctest --verbose -R index_test_py

  # At the head of each Python test file, you can find an instruction
  # describing how to run that test file.

.. HINT::

  As a developer, there is no need to run ``python3 setup.py install``!!!

  All you need is to create a bash script, say ``activate_k2_release.sh``, containing:

    .. code-block:: bash

      #!/bin/bash
      K2_ROOT=/some/path/k2
      export PYTHONPATH=$K2_ROOT/k2/python:$PYTHONPATH
      export PYTHONPATH=$K2_ROOT/build_release/lib:$PYTHONPATH

  To simpily the debug process, we also recommend you to create another bash script,
  e.g., ``activate_k2_debug.sh``, containing:

    .. code-block:: bash

      #!/bin/bash
      K2_ROOT=/some/path/k2
      export PYTHONPATH=$K2_ROOT/k2/python:$PYTHONPATH
      export PYTHONPATH=$K2_ROOT/build_debug/lib:$PYTHONPATH

  To use a release build of k2, run:

    .. code-block:: bash

      source /path/to/activate_k2_release.sh

  To use a debug build of k2, run:

    .. code-block:: bash

      source /path/to/activate_k2_debug.sh

  To check whether you are using a release version or a debug version, run:

    .. code-block:: bash

      python3 -c "import torch; import _k2; print(_k2.__file__)"

  It should print the directory where k2 was built. That is,
  the above output contains a string ``build_release`` or ``build_debug``.

  Alternatively, you can run:

    .. code-block:: bash

      python3 -m k2.version

  You can find the build type in the above output.
