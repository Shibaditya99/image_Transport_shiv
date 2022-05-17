# image_Transport_shiv
Publish and subscribe image data through [ROS]
Before starting any of the image_transport tutorials below, take the time to create a scratch package to work in and manipulate the example code. 
Create a sandbox package with the following dependencies:
    
    $ mkdir -p ~/image_transport_ws/src
    $ cd ~/image_transport_ws/src
    $ source /opt/ros/indigo/setup.bash
    $ catkin_create_pkg learning_image_transport image_transport cv_bridge
    
Make sure to include the correct setup file, in the above example it is for ROS Indigo on Ubuntu and for bash. 
Once you have created the empty package, build it:

    $ cd ~/image_transport_ws
    $ rosdep install --from-paths src -i -y --rosdistro indigo
    $ catkin_make
    $ source devel/setup.bash

The more simplified way to create package from the actual source:

    $ cd ~/image_transport_ws/
    $ git clone https://github.com/ros-perception/image_common.git
    $ mkdir src
    $ ln -s `pwd`/image_common/image_transport/tutorial/ ./src/image_transport_tutorial
    
# Writing a Simple Image Publisher
Here we'll create the publisher node which will continually publish an image.

    #include <ros/ros.h>
    #include <image_transport/image_transport.h>
    #include <opencv2/highgui/highgui.hpp>
    #include <cv_bridge/cv_bridge.h>

    int main(int argc, char** argv)
    {
      ros::init(argc, argv, "image_publisher");
      ros::NodeHandle nh;
      image_transport::ImageTransport it(nh);
      image_transport::Publisher pub = it.advertise("camera/image", 1);
      cv::Mat image = cv::imread(argv[1], CV_LOAD_IMAGE_COLOR);
      cv::waitKey(30);
      sensor_msgs::ImagePtr msg = cv_bridge::CvImage(std_msgs::Header(), "bgr8", image).toImageMsg();

      ros::Rate loop_rate(5);
      while (nh.ok()) {
        pub.publish(msg);
        ros::spinOnce();
        loop_rate.sleep();
      }
    }
    
    
    
# Writing a Simple Image Subscriber
Here we'll create the subscriber node which will display an image topic on screen.

    #include <ros/ros.h>
    #include <image_transport/image_transport.h>
    #include <opencv2/highgui/highgui.hpp>
    #include <cv_bridge/cv_bridge.h>

    void imageCallback(const sensor_msgs::ImageConstPtr& msg)
    {
      try
      {
        cv::imshow("view", cv_bridge::toCvShare(msg, "bgr8")->image);
        cv::waitKey(30);
      }
      catch (cv_bridge::Exception& e)
      {
        ROS_ERROR("Could not convert from '%s' to 'bgr8'.", msg->encoding.c_str());
      }
    }

    int main(int argc, char **argv)
    {
      ros::init(argc, argv, "image_listener");
      ros::NodeHandle nh;
      cv::namedWindow("view");

      image_transport::ImageTransport it(nh);
      image_transport::Subscriber sub = it.subscribe("camera/image", 1, imageCallback);
      ros::spin();
      cv::destroyWindow("view");
    }

# Building your node
Just run:

    catkin_make
