
Description
-----------

This nodelet provides data from a Roboception rc_visard sensor on several ROS topics.

NOTE: The nodelet is based on the GenICam interface of the sensor and requires
a transport layer. This is a shared library with the suffix '.cti'. By default,
a GenTL layer will be installed. Other transport layers can be used by setting
defining their path in the enironment variables GENICAM_GENTL64_PATH for 64 bit
systems and GENICAM_GENTL32_PATH for 32 bit systems.

Configuration
-------------

All parameters marked by * can be reconfigured at runtime.

- device

  The ID of the device, i.e. Roboception sensor.

- gev_access

  The gev_access mode, i.e.:

  - 'control'   Configuration and streaming with the possibility of other
                clients to read GenICam parameters. This is the default.
  - 'exclusive' Exclusive access to the sensor. This prevents other clients to
                read GenICam parameters.
  - 'off'       Switches gev access completely off. The node only streams pose
                information if switched on.

- enable_tf

  If true then the node subscribes to the pose stream and publishes them on tf.
  Default: false

Provided Topics
---------------

The following topics are provided. The nodelet tries to request only images
from the sensor if there is subscriber to the corresponding topic.

- /stereo/left/camera_info (sensor_msgs::CameraInfo)
- /stereo/right/camera_info (sensor_msgs::CameraInfo)

- /stereo/left/image_rect (sensor_msgs::Image, MONO8)
- /stereo/right/image_rect (sensor_msgs::Image, MONO8)

- /stereo/disparity (stereo_msgs::DisparityImage)
- /stereo/disparity_color (sensor_msgs::Image, RGB8, visually pleasing)

- /stereo/depth (sensor_msgs::Image, TYPE_32FC1)
- /stereo/confidence (sensor_msgs::Image, TYPE_32FC1, values between 0 and 1)
- /stereo/error_disparity (sensor_msgs::Image, TYPE_32FC1)
- /stereo/error_depth (sensor_msgs::Image, TYPE_32FC1)

- /stereo/points2 (sensor_msgs::PointCloud2)

For color sensors, the following topics are additionally available:

- /stereo/left/image_rect_color (sensor_msgs::Image, format: RGB8)
- /stereo/right/image_rect_color (sensor_msgs::Image, format: RGB8)


Gazebo Simulation
-----------------

The following topics are provided by the simulated version of the sensor.
- /stereo/left/camera_info (sensor_msgs::CameraInfo)
- /stereo/right/camera_info (sensor_msgs::CameraInfo)

- /stereo/left/image_rect (sensor_msgs::Image, MONO8)
- /stereo/right/image_rect (sensor_msgs::Image, MONO8)

- /stereo/depth (sensor_msgs::Image, TYPE_32FC1)

- /stereo/points2 (sensor_msgs::PointCloud2)

Example Usage in URDF:

    <xacro:rc_visard65 name="rc_1" parent="world" camera_type="color" camera_rate="25Hz">
        <origin xyz="0 0 0" rpy="${-PI/4} 0 ${PI/4}"/>
    </xacro:rc_visard65>

Parameters:

- camera_type:  color or monochrome, depending on the camera type published
- camera_rate:  25Hz, 15Hz or 3Hz as per the documentation, which impacts the resulting image size


Dynamic Parameters
------------------

These parameters can be changed during runtime via dynamic reconfigure:

- camera_fps

  Frames per second that are published by this nodelet. Publishing frames will
  be slowed down depending on this setting. Setting it higher than the real
  framerate of the specific device has no effect.

- camera_exp_auto

  If true, then the exposure time is chosen automatically, up to exp_max as
  maximum. If false, then exp_value is used as exposure time in seconds.

- camera_exp_max

  Maximum exposure time in seconds if exp_auto is true.

- camera_exp_value

  Exposure time in seconds if exp_auto is false.

- camera_gain_value

  Gain factor in decibel if exp_auto is false.

- camera_wb_auto (only for color camera)

  If true, then white balancing is done automatically. If false, then the red
  and blue to green ratios can be chosen manually.

- camera_wb_ratio_red (only for color camera)

  Red to green ratio for color balancing if camera_wb_auto is false.

- camera_wb_ratio_blue (only for color camera)

  Blue to green ratio for color balancing if camera_wb_auto is false.

- depth_quality

  Quality can be "Low", "Medium", "High" and "StaticHigh". Only the first
  letter will be checked, thus specification of "L", "M", "H" or "S" is
  sufficient.

  + StaticHigh quality means computation with 640x480 pixel, limited to 3 Hz
    and accumulation input images. The scene must be static during image
    accumulation! The timestamp of the disparity image is taken from the first
    image that was used for accumulation.
  + High quality means computation with 640x480 pixel.
  + Medium quality means computation with 320x240 pixel.
  + Low quality means computation with 214x160 pixel.

  Default: "High".

- depth_disprange

  Disparity range in pixel, related to the downscaled image at quality=H. The
  range is adapted to the quality.

- depth_fill

  Higher numbers fill gaps with measurments with potentielly higher errors.

- depth_seg

  Maximum size of isolated disparity regions that will be invalidated,
  related to full resolution.

- depth_median

  Performs median filtering with the given window size.

- depth_minconf

  Minimal confidence. All disparities with lower confidence will be set to
  invalid.

- depth_mindepth

  Minimum depth in meter. All disparities with lower depth will be set to
  invalid.

- depth_maxdepth

  Maximum depth in meter. All disparities with higher depth will be set to
  invalid.

- depth_maxdeptherr

  Maximum depth error in meter. All disparities with a higher depth error will
  be set to invalid.

Example
-------

- roslaunch rc_visard_driver.launch

- rosrun rc_visard_driver rc_visard_driver _device:=00_1e_06_32_03_40

- rosrun nodelet nodelet standalone rc_visard_driver _device:=00_1e_06_32_03_40

Build/Installation
------------------

This package depends on libprotobuf-dev, protobuf-compiler and curl
(e.g. libcurl4-gnutls-dev, libcurl4-nss-dev or libcurl4-openssl-dev) which need to be installed in the system.

Some libraries are included as git submodules in this repository
(and themselves include other libs as git submodules).
Hence, before building this package you need to

    git submodule update --init --recursive

Then building then follows the typical ROS catkin workflow.

As an alternative, the cmake build-flow would be something like

    mkdir build && cd build
    cmake -DCATKIN_BUILD_BINARY_PACKAGE="1" -DCMAKE_INSTALL_PREFIX="/opt/ros/$ROS_DISTRO" -DCMAKE_PREFIX_PATH="/opt/ros/$ROS_DISTRO" -DCMAKE_BUILD_TYPE=Release ..
    make
