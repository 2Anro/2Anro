import paho.mqtt.client as mqtt

# MQTT broker details
broker_address = "mqtt.example.com"
broker_port = 1883
broker_username = "your_username"
broker_password = "your_password"

# Topic to control the light bulb
light_topic = "home/living_room/light"

# Callback function for MQTT connection
def on_connect(client, userdata, flags, rc):
    print("Connected to MQTT broker")
    client.subscribe(light_topic)

# Callback function for receiving MQTT messages
def on_message(client, userdata, msg):
    if msg.topic == light_topic:
        if msg.payload.decode() == "on":
            # Code to turn the light on
            print("Light turned on")
        elif msg.payload.decode() == "off":
            # Code to turn the light off
            print("Light turned off")

# Create an MQTT client instance
client = mqtt.Client()

# Set MQTT authentication details
client.username_pw_set(broker_username, broker_password)

# Set MQTT callbacks
client.on_connect = on_connect
client.on_message = on_message

# Connect to the MQTT broker
client.connect(broker_address, broker_port)

# Start the MQTT loop
client.loop_start()

# Main program loop
while True:
    command = input("Enter command (on/off): ")
    if command.lower() == "on":
        # Publish MQTT message to turn the light on
        client.publish(light_topic, "on")
    elif command.lower() == "off":
        # Publish MQTT message to turn the light off
        client.publish(light_topic, "off")
