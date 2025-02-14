# Optimization of recombinant antibody production in E. coli using dual GFP promoter construct, a blue LED and a camera
### Anttoni Korkiakoski, Sami Oksanen, Tuomas Huovinen

Additional information for the assembly of GFP biosensor described in Korkiakoski et al. (2025) Optimization of recombinant antibody production in E. coli using dual GFP promoter construct, a blue LED and a camera.

# Python script for running the sensor:

````python
# Libraries
from picamera import PiCamera
import time
from fractions import Fraction
import datetime
import RPi.GPIO as GPIO
from PIL import Image
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd

# Camera setup
camera = PiCamera(framerate=Fraction(1,6))
camera.resolution = (1600, 1200)
camera.shutter_speed = 1000000    # max 6M µs for Cam V2.1 
camera.iso = 800                         # max: ISO800 
camera.exposure_mode = 'off'          # does not make automatic gain adjustments

ledpin = 18             
GPIO.setmode(GPIO.BOARD)       # use PHYSICAL GPIO Numbering
GPIO.setup(ledpin, GPIO.OUT)     # set RGBLED pins to OUTPUT mode

#Image processing
ts = []
gys = []
bys = []
rys = []
xs = []
names = [] #added 220916TH

plt.ion()
figure, ax = plt.subplots(figsize=(8,6))
plt.xlabel("Time (h)",fontsize=18)
plt.ylabel("GFP fluorescence (pixel sum)",fontsize=18)
plt.title("Dynamic Plotting",fontsize=25)

i=1
time_1 = time.time()
print("here we are")
try:
    while True:
        GPIO.output(ledpin, GPIO.HIGH) #BLUE LED ON TO MEASURE GFP GREEN 
        time.sleep(2)
        #COMMON TIME
        now = datetime.datetime.now()
        timestamp = now.strftime("%B_%d_%H%M")
        time_2 = time.time()
        time_d = float((time_2 - time_1)/3600)
        hours = "{:.2f}".format(time_d)

        #PROCESS RED, GREEN AND BLUE
        filename = "img_" + str(i) + "_" + timestamp + "_Green" + '.jpg'
        camera.capture(filename)
        print(filename)
        img = Image.open(filename).convert('RGB')
        img.load()
        sequence_of_Gpixels = np.asarray(img.getchannel(1), dtype="int32")
        sequence_of_Bpixels = np.asarray(img.getchannel(2), dtype="int32")
        sequence_of_Rpixels = np.asarray(img.getchannel(0), dtype="int32")
        sumgreen = sequence_of_Gpixels.sum()
        sumblue = sequence_of_Bpixels.sum()
        sumred = sequence_of_Rpixels.sum()
        time.sleep(2)
        GPIO.output(ledpin, GPIO.LOW) #BLUE LED OFF
            
         #ADD PIXEL VALUES TO LIST
        ts.append(timestamp)
        gys.append(sumgreen)
        bys.append(sumblue)
        rys.append(sumred)
        xs.append(time_d)
                
        df = pd.DataFrame({"Time" : xs, "SumG" : gys, "SumB" : bys, 
                           "SumR" : rys, "Stamp" : ts}).sort_values(by = "Time")
        line1, = ax.plot(df.Time, df.SumG, "g-")
        line1.set_xdata(df.Time)
        line1.set_ydata(df.SumG)
        line2, = ax.plot(df.Time, df.SumB, "b-")
        line2.set_xdata(df.Time)
        line2.set_ydata(df.SumB)
        line3, = ax.plot(df.Time, df.SumR, "r-")
        line3.set_xdata(df.Time)
        line3.set_ydata(df.SumR)
                      
        figure.canvas.draw()
        figure.canvas.flush_events()    
        print(df)
        time.sleep(296)
        i+=1
     
except KeyboardInterrupt: #Press Ctrl+C to interrupt
    timestamp2 = now.strftime("%Y_%B_%d_%H%M")
    df.to_csv('output_' + timestamp2 + '.csv')
    camera.close()
    GPIO.cleanup()
    pass
````
