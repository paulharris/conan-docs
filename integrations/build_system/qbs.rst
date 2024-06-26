.. _qbs:


qbs
____

.. warning::

    This is a **deprecated** feature. Please refer to the :ref:`Migration Guidelines<conan2_migration_guide>`
    to find the feature that replaced this one.

Conan provides a **qbs** generator, which will generate a ``conanbuildinfo.qbs`` file that can be used for your
qbs builds.

Add ``conanbuildinfo.qbs`` as a reference on the project level and a Depends item with the name ``conanbuildinfo``:

**yourproject.qbs**

.. code-block:: qbs

   import qbs

   Project {
      references: ["conanbuildinfo.qbs"]
      Product {
           type: "application"
           consoleApplication: true
           files: [
               "conanfile.txt",
               "main.cpp",
           ]
           Depends { name: "cpp" }
           Depends { name: "ConanBasicSetup" }
      }
   }

This will include the product called ``ConanBasicSetup`` which holds all
the necessary settings for all your dependencies.

If you'd prefer to manually add each dependency, just replace
``ConanBasicSetup`` with the dependency you would like to include.
You may also specify multiple dependencies:

**yourproject.qbs**

.. code-block:: qbs

   import qbs

   Project {
      references: ["conanbuildinfo.qbs"]
      Product {
           type: "application"
           consoleApplication: true
           files: [
               "conanfile.txt",
               "main.cpp",
           ]
           Depends { name: "cpp" }
           Depends { name: "catch" }
           Depends { name: "Poco" }
      }
   }

.. seealso::

    Check the :ref:`Reference/Generators/qbs<qbs_generator>` section for get more details.
