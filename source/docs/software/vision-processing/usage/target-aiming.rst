Use Case: Aiming to a Target
============================

This use case is one of the most common use cases, but also very simple. It can be very useful when we need to either shoot into a target like in Infinite Recharge, or when you need to place game elements into a scoring location accurately, like in Deep Space.
Almost every single season that has had vision targets, aiming to a target can be very useful.

.. note:: When working on your vision solution, aiming to a target benefits from higher speed vision, much more than higher accuracy/resolution.

Requirements
------------

To aim to our target, we need our yaw (horizontal / x axis offset) to the target. In order to aim we do not need the exact angle, just a value to corresponding to our angle to the target.
In this article we assume there is a `NetworkTable` entry containing a number and named `yaw` under the table `camera` that is consonantly being fed the current yaw.

Additionally you will need to have some code for driving, it is assumed here that you use one of WPILib's drive classes such as ``DifferentialDrive`` (`Java <https://first.wpi.edu/FRC/roborio/release/docs/java/edu/wpi/first/wpilibj/drive/DifferentialDrive.html>`__, `C++ <https://first.wpi.edu/FRC/roborio/release/docs/cpp/classfrc_1_1DifferentialDrive.html>`__). For more info on using these classes take a look at :ref:`<docs/software/actuators/wpi-drive-classes:Using the WPILib Classes to Drive your Robot>`.
This article calls this object `drive`(Java)/'Drive'(C++).


Implementation
--------------

The basic principle behind aiming to a target is to command the drivetrain to turn based on how big our error is from the target.

To align we need some kind of controller to react to the error to the target, in this article we will be using a ``PIDController`` (`Java <>`__, `C++ <>`__), for more information about controllers and PID controllers, take a look at :ref:'this article<docs/software/advanced-controls/introduction/control-system-basics:Control System Basics>' and the ones that follow it.
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

Tank Drive
^^^^^^^^^^

This is the easiest method to implement if your not using WPILib's drive classes, since all this means is that we give a certain power to each side of the drivetrain.

The implementation itself is quite simple, first of all we get the yaw from our `NetworkTable`, then all we have to do is get the power to command to command from the controller, and give each side opposing power to make the robot turn.

.. tabs::

   .. code-tab:: java

     double yaw = table.getEntry("yaw").getDouble();
     double power = MathUtil.clamp(controller.calculate(yaw), -1, 1);

     //If the robot turns the wrong direction, simply swap the 2 sides of the drivetrain to turn the robot in the opposite direction.
     drive.tankDrive(power, -power);

   .. code-tab:: c++

     double yaw = 0;

Arcade Drive
^^^^^^^^^^^^

This method is a bit harder to implement if not using WPILib's drive classes, but will make implementing additional functionality easier.

If using WPILib's drive classes the implementation is also quite easy, once again we need to get our yaw from our `NetworkTable`, then get the power in which to turn the robot, and use arcade drive to command the robot to turn at that rate.

.. tabs::

   .. code-tab:: java

     double yaw = table.getEntry("yaw").getDouble();
     double power = MathUtil.clamp(controller.calculate(yaw), -1, 1);

     //If the robot turns the wrong direction, simply negate the power, causing the robot to turn the opposite direction.
     drive.arcadeDrive(0, power);

Now, this should be enough for most use cases, simply tune the P value, by raising/lowering it until you get a reasonable response time with minimal oscillations. To get a full look into please refer to :ref:`<docs/software/advanced-control/introduction/turning-pid-controller:Tuning a PID Controller>`

Additional Functionality
------------------------


