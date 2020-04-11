Use Case: Aiming to a Target
============================

This use case is one of the most common use cases, but also very simple. It can be very useful when we need to either shoot into a target like in Infinite Recharge, or when you need to place game elements into a scoring location accurately, like in Deep Space.
Almost every single season that has had vision targets, aiming to a target can be very useful.

.. note:: When working on your vision solution, aiming to a target benefits from higher speed vision, much more than higher accuracy/resolution.

Requirements
------------

To aim to our target, we need our yaw (horizontal / x axis offset) to the target. In order to aim we do not need the exact angle, just a value to corresponding to our angle to the target.
In this article we assume there is a ``NetworkTable`` entry containing a number and named ``yaw`` under the table ``camera`` that is consonantly being fed the current yaw.

Additionally you will need to have some code for driving, it is assumed here that you use one of WPILib's drive classes such as ``DifferentialDrive`` (`Java <https://first.wpi.edu/FRC/roborio/release/docs/java/edu/wpi/first/wpilibj/drive/DifferentialDrive.html>`__, `C++ <https://first.wpi.edu/FRC/roborio/release/docs/cpp/classfrc_1_1DifferentialDrive.html>`__). For more info on using these classes take a look at :ref:`docs/software/actuators/wpi-drive-classes:Using the WPILib Classes to Drive your Robot`.
This article calls this object ``drive`` (Java)/``Drive`` (C++).


Implementation
--------------

The basic principle behind aiming to a target is to command the drivetrain to turn based on how big our error is from the target.

To align we need some kind of controller to react to the error to the target, in this article we will be using a ``PIDController`` (`Java <https://first.wpi.edu/FRC/roborio/release/docs/java/edu/wpi/first/wpilibj/PIDController.html>`__, `C++ <https://first.wpi.edu/FRC/roborio/release/docs/cpp/classfrc2_1_1PIDController.html>`__), for more information about controllers and PID controllers, take a look at :ref:`docs/software/advanced-controls/introduction/control-system-basics:Control System Basics` and the ones that follow it.
Some of these example actually only use the controller to implement a P controller or a PD controller, but we will still use the ``PIDController`` WPILib class, with gains of 0 for the unused components of the controller.

There are a number of ways to implement the driving itself, we will cover a few of them here.

First of all both methods need the actual ``PIDController`` object, so we will start by making it, in this example we have a simple P controller.

.. tabs::

   .. code-tab:: java

     //The gains used here are arbitrary and need to be tuned for your specific robot
     PIDController controller = new PIDController(0.04, 0, 0);

   .. code-tab:: c++

     //The gains used here are arbitrary and need to be tuned for your specific robot
     frc2::PIDController(0.04, 0, 0);

Then to the drive implementation.

Tank Drive Implementation
^^^^^^^^^^^^^^^^^^^^^^^^^

This is the easiest method to implement if your not using WPILib's drive classes, since all this means is that we give a certain power to each side of the drivetrain.

The implementation itself is quite simple, first of all we get the yaw from our ``NetworkTable``, then all we have to do is get the power to command to command from the controller, and give each side opposing power to make the robot turn.

.. note:: Remember, this code needs to be run periodically, meaning it has to be either in a ``periodic`` method for a TimedRobot, or in a command's ``execute`` method. If you are running your loops at a speed other than once per 20ms (50hz), make sure to change the PIDController as needed.

.. tabs::

   .. code-tab:: java

     double yaw = table.getEntry("yaw").getDouble();
     double power = MathUtil.clamp(controller.calculate(yaw), -1, 1);

     //If the robot turns the wrong direction, simply swap the 2 sides of the drivetrain to turn the robot in the opposite direction.
     drive.tankDrive(power, -power);

   .. code-tab:: c++

     double yaw = 0;

Arcade Drive Implementation
^^^^^^^^^^^^^^^^^^^^^^^^^^^

This method is a bit harder to implement if not using WPILib's drive classes, but will make implementing additional functionality easier.

If using WPILib's drive classes the implementation is also quite easy, once again we need to get our yaw from our ``NetworkTable``, then get the power in which to turn the robot, and use arcade drive to command the robot to turn at that rate.

.. note:: Remember, this code needs to be run periodically, meaning it has to be either in a ``periodic`` method for a TimedRobot, or in a command's ``execute`` method. If you are running your loops at a speed other than once per 20ms (50hz), make sure to change the PIDController as needed.

.. tabs::

   .. code-tab:: java

     double yaw = camera.getEntry("yaw").getDouble();
     double power = MathUtil.clamp(controller.calculate(yaw), -1, 1);

     //If the robot turns the wrong direction, simply negate the power, causing the robot to turn the opposite direction.
     drive.arcadeDrive(0, power);

Now, this should be enough for most use cases, simply tune the P value, by raising/lowering it until you get a reasonable response time with minimal oscillations. To get a full look into please refer to :ref:`docs/software/advanced-controls/introduction/tuning-pid-controller:Tuning a PID Controller`

Additional Functionality
------------------------

This section covers a few changes that can be made to add additional functionality on top of just aiming to a target.
These can come useful in various seasons in many different ways.

Driving While Aligning
^^^^^^^^^^^^^^^^^^^^^^

The most common one is probably driving forward/backwards with a joystick.
This comes in very useful in seasons like 2019, where on top of aligning with the target, we also need to reach it, while staying aligned.
These examples use a joystick named ``joystick`` (Java)/``Joystick`` (C++)

Driving While Aligning With Tank Drive
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Implementation in tank drive can be a bit confusing, to make it easy to read, we store the left and right powers in their own variables.

.. tabs::

   .. code-tab:: java

     double forwardPower = joystick.getY();

     //Once again if you are turning the wrong way, swap the 2 sides
     double leftPower = MathUtil.clamp(power + forwardPower, -1, 1);
     double rightPower = MathUtil.clamp(-power + forwardPower, -1, 1);

     drive.tankDrive(leftPower, rightPower);

Driving While Aligning With Arcade Drive
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Implementation in tank drive is very simple, we simply have to change a single line.

.. tabs::

   .. code-tab:: java

     drive.arcadeDrive(joystick.getY(), power);

Seeking
^^^^^^^

Seeking comes useful, when we need to aim at a target, but we can't be sure we are currently looking at it. What we do is just make the robot turn in place until it sees a target.
Because we need to know when a target is visible, this requires another value from our vision processing, a ``hasTarget`` value.

The implementation itself is the same for both tank drive and arcade drive, because we are only modifying the power given to turn.
Simply replace the line where we set the ``power`` variable with this.

.. tabs::

   .. code-tab:: java

     //You can change this value to negative to turn the other direction if desired.
     //Additionally, you should adjust this value to fit your robot, and your needs for turning speed and stability.
     double power = 0.5;
     if (camera.getEntry("hasTarget").getBoolean()) {
       power = MathUtil.clamp(controller.calculate(yaw), -1, 1);
     }
