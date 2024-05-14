import time
import numpy as np
import thingspeak
import time
import random
import smbus
import RPi.GPIO as GPIO

API_KEY = 'OFSACL68JYHJNUAA'
address = 0x04
address2 = 0x05

canal = thingspeak.Channel(2524802, API_KEY)

class PID:
    def __init__(self, P, I, D, N, setpoint):
        self.Kp = P
        self.Ki = I
        self.Kd = D
        self.N = N
        self.setpoint = setpoint
        self.prev_error = 0
        self.integral = 0

    def compute(self, feedback):
        error = self.setpoint - feedback
        self.integral += error
        derivative = error - self.prev_error
        output = self.Kp * error + self.Ki * self.integral + self.Kd * derivative
        output = np.clip(output, -1, 1)  # Limitar la salida entre -1 y 1
        self.prev_error = error
        return output

# Función para simular el proceso de control
def simulate_control(pid):
    feedback = 0  # Valor inicial del sensor
    while True:
        # Simular proceso de control, aquí deberías leer el valor del sensor
        # y actualizar la variable feedback
        # En este ejemplo, simplemente vamos a sumar un valor aleatorio
        feedback += np.random.uniform(-0.5, 0.5)
        output = pid.compute(feedback)
        print("Feedback:", feedback, "Output:", output)
        time.sleep(0.1)

def recive_data():
    bus = smbus.SMBus(1)
    temperature = bus.read_byte(address)
    voltage = temperature * (5.0 / 255)
    temperatureC = voltage
    #temperature_float= float.fromhex(''.join(format(byte,'02x') for byte in temperature))
    return (temperature*500) / 1023

def recive_data2():
    bus = smbus.SMBus(1)
    viscosity = bus.read_byte(address2)
    
    return viscosity

def write_data(value):
    bus.write_byte(address2, value)
    # Optionally, you can add a delay here if needed
    time.sleep(0.1)


while True:
    pid = PID(P=-1.40, I=-0.92, D=-0.043, N=12.13, setpoint=0)
    write_data(int(24))
    try:     
        data = recive_data2()
        r = pid.compute(data)
        print("Dato: {}".format(data))
        canal.update({2: data})
        if r > 0:
            write_data(int(56))
        else:
            write_data(int(36))
        time.sleep(1)
    except IOError as e:
        print(e)
        time.sleep(1)
    finally:
        GPIO.cleanup()    
    
    #valor = random.randint(0,100)
    #
    #print(valor)
    #time.sleep(1)
