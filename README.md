# link

import thingspeak
import time
import random
import smbus
import RPi.GPIO as GPIO

API_KEY = 'XDT3Z7N5FSEP0YMI'
address = 0x04

canal = thingspeak.Channel(2524802, API_KEY)

def receive_data():
    bus = smbus.SMBus(1)
    temperature = bus.read_byte(address)
    voltage = temperature * (5.0 / 255)
    temperatureC = voltage
    return (temperature * 500) / 1023

def start_button_callback(channel):
    global running
    running = True

def stop_button_callback(channel):
    global running
    running = False

GPIO.setmode(GPIO.BOARD)
pin_led = 11
pin_start_button = 13
pin_stop_button = 15

GPIO.setup(pin_led, GPIO.OUT)
GPIO.setup(pin_start_button, GPIO.IN, pull_up_down=GPIO.PUD_UP)
GPIO.setup(pin_stop_button, GPIO.IN, pull_up_down=GPIO.PUD_UP)

GPIO.add_event_detect(pin_start_button, GPIO.FALLING, callback=start_button_callback, bouncetime=300)
GPIO.add_event_detect(pin_stop_button, GPIO.FALLING, callback=stop_button_callback, bouncetime=300)

running = False

try:
    while True:
        if running:
            try:
                data = receive_data()
                if data >= 30:
                    GPIO.output(pin_led, GPIO.HIGH)
                else:
                    GPIO.output(pin_led, GPIO.LOW)
                print("Dato: {}".format(data))
                canal.update({1: data})
                time.sleep(1)
            except IOError as e:
                print(e)
                time.sleep(1)
        else:
            GPIO.output(pin_led, GPIO.LOW)
            time.sleep(0.1)

except KeyboardInterrupt:
    pass
finally:
    GPIO.cleanup()
