Aiming to a Target
==================

Here we'll talk about how to aim to our vision target.
This can be useful when we need to either shoot into a target like in Infinite Recharge, and when you need to place game elements into a scoring location accurately, like Deep Space.

Aiming at a target both very simple and very useful, and can be useful in most FRC games.

Requirements
------------

In order to aim we need yaw from our target (horizontal / x offset), for this implementation the actual angle is not necessary, just some kind of value corresponding to our angle to target.
In the follow code examples it is assumed that the value is a double named ``yaw`` with this value like in the following code snippet

.. tabs::

  .. code-tab:: java

    double yaw = getTargetYaw();

  .. code-tab:: c++

It is also assumed that you are updating this value once per loop iteration yourself.

Additionally you need to have some code for driving, it is assumed here that you use one of WPILib's drive classes such as ``DifferentialDrive`` (`Java <https://first.wpi.edu/FRC/roborio/release/docs/java/edu/wpi/first/wpilibj/drive/DifferentialDrive.html>`__, `C++ <https://first.wpi.edu/FRC/roborio/release/docs/cpp/classfrc_1_1DifferentialDrive.html>`__).
The tutorial call this object ``drive``, like in this example.

.. tabs::

  .. code-tab:: java

    DifferentialDrive drive = new DifferentialDrive(leftMotor, rightMotor);

  .. code-tab:: c++

Implementation
--------------

The basic principle behind this, is we command the drivetrain to turn based on how big our error is from the target.
There are a few methods to do this, we will cover one method, but will also explain a bit about other ways.

For aligning with the target we use a ``PIDController`` (`Java <>`__, `C++ <>`__), for more info about it take a look at :ref:`/docs/software/advanced-control/controllers/pidcontroller`.

.. tabs::

  .. code-tab:: java

    //The gains used here are arbitrary and need to be tuned for your specific robot
    PIDController controller = new PIDController(0.04, 0, 0);

  .. code-tab:: c++

Then we command each side of the drivetrain with opposite powers, using the tank drive method of the drive class

.. tabs::

  .. code-tab:: java

    double power = MathUtil.clamp(controller.calculate(yaw), -1, 1);

    //If the robot turns the wrong direction, simply swap the 2 sides of the drivetrain
    drive.tankDrive(power, -power);

  .. code-tab:: c++

Now, this should be enough for most use cases, simply modify the PID values.
To tune the PID values start with a low value of P, and start slowly increasing until it starts oscillating, then either lower the P value or start increasing D. You can try both and see which one works best for you.

Alternate Implementation
------------------------

In some cases the robot won't actually align itself, in those cases
