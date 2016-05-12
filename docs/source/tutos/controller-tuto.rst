Create your first OROCOS controller
###################################

Using ``lwr_create_pkg``
------------------------

Using the lwr_project_creator utility, we can generate a (very) simple controller for our robot.

Generate the controller
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cd ~/
    lwr_create_pkg my_controller -C MyController


Build the controller
~~~~~~~~~~~~~~~~~~~~

.. image:: /_static/run_example.png

.. code-block:: bash

    cd ~/my_controller
    mkdir build ; cd build ; cmake ..
    make

    # add the controller to the ros package path
    source devel/setup.bash

Launch it :

.. code-block:: bash

    roslaunch my_controller run.launch sim:=true


..image:: /_static/run_example2.png

.. note:: Now you can **delete** this controller and create your own on your ``catkin worskpace`` with a better name :)

General note on controllers
---------------------------

This generated controller is "kuka-lwr-free" and as a general rule you should avoid to include kuka-lwr stuff in your code.

You should just see the robot as an ***Effort interface***,i.e, you should only listen to the current state (q,qdot) and publish torques.

**Recommended** ports are :

.. code-block:: bash

    Robot input / Controller output:   - JointTorqueCommand
                                       - (JointPositionCommand)

    Robot Output / Controller intput:  - JointPosition
                                       - JointVelocity
                                       - (JointTorque)



The complete list of LWR ports :

.. code-block:: bash

    curl --silent https://raw.githubusercontent.com/kuka-isir/lwr_hardware/indigo-devel/lwr_fri/src/FRIComponent.cpp  | grep -oP 'addPort\( *\"\K\w+'


.. code-block:: bash

    CartesianImpedanceCommand
    CartesianWrenchCommand
    CartesianPositionCommand
    JointImpedanceCommand
    JointPositionCommand
    JointTorqueCommand          --> To Controller
    toKRL                       --> KRL Tool
    KRL_CMD                     --> KRL Tool
    fromKRL                     --> KRL Tool
    CartesianWrench
    RobotState                  --> KRL Tool
    FRIState                    --> KRL Tool
    JointVelocity               --> To Controller
    CartesianVelocity
    CartesianPosition
    MassMatrix
    Jacobian
    JointTorque                 --> To Controller
    JointTorqueAct
    GravityTorque
    JointPosition               --> To Controller
    JointPositionFRIOffset


The controller uses `rtt_ros_kdl_tools::ChainUtils <https://github.com/kuka-isir/rtt_ros_kdl_tools/blob/master/src/chain_utils.cpp/>`_ to create an "arm" object.
This arm loads the robot_description from the ROS parameter server (you can use the provided launch file that helps you start everything), then create a few KDL chain, solvers etc to compute forward kinematics, jacobians etc.

Functions available can be found `here <https://github.com/kuka-isir/rtt_ros_kdl_tools/blob/master/include/rtt_ros_kdl_tools/chain_utils.hpp/>`_.

Inverse Kinematics in not included in ChainUtils as there's not "perfect" solution for every problem. Meanwhile a few approches exists :

- All the chainik in `KDL <https://github.com/orocos/orocos_kinematics_dynamics/tree/master/orocos_kdl/src/>`_
- `Trac IK <https://bitbucket.org/traclabs/trac_ik.git/>`_ , same interface as KDL
- Simple Jacobian transpose method
    * Write into the kuka lwr CartesianPositionCommand port some cartesian position increments, but it is not recommended as it is kuka specific.
    * Instead, use the ChainUtils arm object to compute :math:`Jt.(K(q^{*}-q) + D(\dot{q^{*}}-\dot{q}))`

Make you life easier with Kdevelop
----------------------------------

Kdevelop is a nice IDE that supports cmake natively.
To install it, just type ``sudo apt-get install kdevelop``, then in a **terminal**, type ``kdevelop``.

.. tip::

        To enable sourcing the bashrc from the Ubuntu toolbar, ``sudo nano /usr/share/applications/kde4/kdevelop.desktop``
        and replace ``Exec=kdevelop %u`` by ``Exec=bash -i -c kdevelop %u``.



Import a catkin/CMake project
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Click on ``Project --> Open/Import Project...``

.. image:: /_static/kdev-import.png

Select the ``CMakeLists.txt`` inside your project.

.. image:: /_static/kdev-import-cmake.png


Select **you project** as the root directory.

.. image:: /_static/kdev-import-root.png

Correct the ``Build Directory`` if necessary, and if you've already built with ``catkin build``, you should see ``Using an already created build directory``.

.. image:: /_static/kdev-import-end.png

.. warning::

        Make sure the ``Build Directory`` is set to ``/path/to/catkin_ws/build/my_project``.

Click on ``finish`` and you're done import your project.

.. image:: /_static/kdev-import-finish.png



Build your project
~~~~~~~~~~~~~~~~~~

On the vertical left panel, click on ``Projects`` and you'll see the list of your currently opened projects. Yours should appear here afer a few seconds.

.. image:: /_static/kdev-editor.png

You can check also in the ``Build Sequence`` that your project appears.
To build, click on ``build`` :)

.. image:: /_static/kdev-build.png

.. note::

    Cliking on ``build`` is equivalent to calling :

    .. code::

        cd /path/to/catkin_ws
        catkin build my_super_project