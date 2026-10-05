# MQTT

So far, we have used Wi-Fi to connect the ESP32-C5 to a network and communicate directly with a website. This works well when a device needs to communicate with a specific server, but IoT systems often need a different approach.

Imagine you have several devices in your home. A device with a temperature sensor could periodically send temperature readings. A phone application could display those readings. Another device could use the same readings to control a fan.

One way to build this would be to make every device communicate directly with every other device. As the number of devices grows, this quickly becomes complicated. 

MQTT simplifies this by allowing devices to communicate through a central broker instead of directly with each other. The broker is a server that receives messages and delivers them to devices that are interested in them.

MQTT is a lightweight messaging protocol designed for IoT and other environments where devices may have limited resources or network bandwidth. It uses a **publish/subscribe** model.

<figure >
  <img 
    class="" 
    src="./images/MQTT Overall flow and architecture.jpg" 
    alt="MQTT Publish / Subscribe Architecture"
  >
  <figcaption style="text-align: center; padding-top: 5px">
    MQTT Publish / Subscribe Architecture
  </figcaption>
</figure>

Instead of sending a message directly to another device, a device publishes a message to a `topic`. A `topic` is a name that identifies a category of messages. Devices that want to receive those messages subscribe to the topic. The MQTT broker then delivers the messages to those subscribers.
