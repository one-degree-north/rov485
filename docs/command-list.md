# Command List
All values are in this format:
- [cmd] - [name]
	- param: [param]
	- data field: explanation of data field usage
	- **returns** if it returns something other than 0xFF (OK)
	- explanation of the command

### Forward Communication

- 0x00 - test
	- param: must be 0xF0 (system)
	- data field: None
	- **returns** with the return 0x00 test packet.
	- used for testing.
- 0x0F - HALT
	- param: Any
	- data field: None
	- halts all motors and servos. 
- 0x10 - setMotorMicroseconds
	- param: hardware motor (0-3, either 0x00-0x03 or '0'-'3')
	- data field: first two bytes are a `uint16_t`, range `PWM_MIN` to `PWM_MAX`
	- sets the motor's exact PWM microseconds using Arduino's `writeMicroseconds` function.
- 0x12 - setMotorCalibrated
	- param: hardware motor (0-3, either 0x00-0x03 or '0'-'3')
	- data field: first byte is a `int8_t`, range -100-100
	- sets the motor's speed, according to its calibration.`PWM_MIN`, and `PWM_MAX`
- 0x13 - setMotorCalibration
	- param: hardware motor (0-3, either 0x00-0x03 or '0'-'3')
	- data field: first two bytes are a `uint16_t`, range 0-4000
	- sets the motor's calibration value, where 1000 is normal, 500 is half speed, and 2000 is double speed.
- 0x30 - getIMU
	- param: must be 0x15 (accelerometer), 0x16 (gyroscope), 0x18 (linear acceleration), 0x19 (orientation)
	- data field: None
	- **returns** with three packets representing the X, Y, Z of the specified device. (0x3A or 0x3C)
	- fetches the current IMU 
- 0x33 - setAccelSettings (Deprecated)
	- param: must be 0x15
	- data field: first byte is range (0-3 or '0'-'3'), second byte is divisor (0-255)
	- sets the accelerometer divisor and range for current operation.
- 0x34 - setGyroSettings (Deprecated)
	- param: must be 0x16
	- data field: first byte is range (0-3 or '0'-'3'), second byte is divisor (0-255)
	- sets the gyroscope divisor and range for current operation.
- 0x35 - getIMUSettings
    - param: Any
    - data field: None
    - returns wtih the 0x3E packet.
- 0x40 - getVoltageAndTemperature
	- param: must be 0x17
	- data field: None
	- **returns** with packet 0x44 (temp & voltage)
- 0x43 - setVoltageCalibration
	- param: must be 0x17
	- data field: four bytes are a single-precision `float`, representing the calibration value (default 7.55)
	- sets the voltage calibration value.
- 0x50 - setAutoReport
	- param: must be 0x15 (accel), 0x16 (gyro), 0x17 (voltage/temp), 0x18 (linear acceleration), 0x19 (orientation)
	- data field: first byte is 0 or any other value, second and third bytes are a `uint16_t` specifying how often (in milliseconds) to send packets. 
	- if first byte is not 0x00, autoReport is turned on for the specified device, and the system will attempt to report once every specified number of milliseconds.
- 0x51 - setFeedback
	- param: must be 0x01 (system)
	- data field: first byte is 0x00 or any other value.
	- if first byte is 0x00, feedback is turned off, otherwise feedback is enabled. this prevents the system from sending 0x0F OK signals.

### Return Communication
- 0x00 - test
	- param: 0xF0 (system)
	- data field: first byte returns the version of the code currently running. next 3 bytes contain the word "pog".
- 0x0A - OK
	- param: 0x00 (fail) or 0xFF (success)
	- data field: None
	- returned as a success message when any command with "set" is invoked.
- 0x1C - motor_status
    - param: `uint8_t` representing  the range of motor 4's motion (1010-1660)
    - data field: four `int8_t`s representing the most recently set % of each motor
- 0x3A - accelerometer
	- param: 0x00 (X), 0x30 (Y), 0x60 (Z)
	- data field: four bytes are a single-precision `float`, representing the current value of that axis of the accelerometer.
- 0x3B - linear acceleration
	- param: 0x00 (X), 0x30 (Y), 0x60 (Z)
	- data field: four bytes are a single-precision `float`, representing the current value of that axis of linear acceleration.
- 0x3C - gyroscope
	- param: 0x00 (X), 0x30 (Y), 0x60 (Z)
	- data field: four bytes are a single-precision `float`, representing the current value of that axis of the gyroscope.
- 0x3D - orientation
	- param: 0x00 (X), 0x30 (Y), 0x60 (Z)
	- data field: four bytes are a single-precision `float`, representing the current value of that axis of absolute orientation.
- 0x3E - imu calibration
    - param: Any
    - data field: each byte represents one calibration value reported by the BNO055, in the order system-gyro-accel-mag.
- 0x44 - voltage/temperature
	- param: 0x17
	- data field: first two bytes are a `uint16_t` representing 100 times the current temperature in Celsius, the final two bytes are a `uint16_t` representing 100 times the current voltage. This is used since there is no such thing (or need) for a 16-bit floating point value.
