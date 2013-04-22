# Augmented Reality View for Android #

----------


**AugmentedRealityView** project created by the mobile development team at [raw engineering](http://raweng.com "raw engineering."). This project can be integrated into any other app, needing AR View.
It is available on [Github](https://github.com/Bhide/AugmentedRealityView.git).

[Augmented reality](http://en.wikipedia.org/wiki/Augmented_reality) (AR) is a live, direct or indirect, view of a physical, real-world environment whose elements are augmented by computer-generated sensory input such as sound, video, graphics or GPS data. It is related to a more general concept called mediated reality, in which a view of reality is modified (possibly even diminished rather than augmented) by a computer. As a result, the technology functions by enhancing one’s current perception of reality.

## Software and Algorithms ##

A key measure of AR systems is how realistically they integrate augmentations with the real world. The software must derive real world coordinates, independent from the camera, from camera images.
Our app for Augmented Reality View uses android phone sensors and certain physics concepts for calculating the Point Of Interest (POIs).
The Camera Angle of View and Android's magnetic and accelerometer sensors are considered for calculating the azimuth (yaw), pitch and roll positions.

Android's `SurfaceView` is used to render the device camera and put markers onto it.<br><br>
![image](https://dl.dropboxusercontent.com/u/2906868/ARView.png)
![image](https://dl.dropboxusercontent.com/u/2906868/places_near_list.png)
<br>**figure : Augmented Reality View for Android**

<br>Two main classes that comprise this View are:<br>
1. RadarView.<br>
2. DataView.

<br>`RadarView` class takes care of the Radar shown on the left top corner. It shows the no. of points that are in the neighbourhood. The points in the 'V' notch is the area that is facing the camera.
The points in the V notch are in the '[Angle of View](http://en.wikipedia.org/wiki/Angle_of_view)' of camera.

`DataView` class takes care of all the data and markers that are shown on the phone screen. Just pass the current location co-ordinates (latitude,longitude) in variables and the `ArrayList` of locations of places you have to plot.<br><br>

*Code snippet:*<br>
public void init(int widthInit, int heightInit, android.hardware.Camera camera, DisplayMetrics displayMetrics, RelativeLayout rel) {



try {

			locationMarkerView = new RelativeLayout[latitudes.length];

			layoutParams = new RelativeLa.......... ... . .

                        .......... . ........... .. ... .

    }
}

<br>The `init ()` method allows you to initialize the variable sets, that is used for calculating the necessary values for plotting the markers. <br>
ex. `onClickListeners` of markers, camera view angles, screen dimensions.<br>
*Code snippet :*

public void draw(PaintUtils dw, float yaw, float pitch, float roll) {


         this.yaw = yaw;

		 this.pitch = pitch;

		 this.roll = roll;



}

The `draw ()` method paints the Radar View on the left top corner.<br>
*Code snippet :*

void radarText(PaintUtils dw, String txt, float x, float y, boolean bg, boolean isLocationBlock, int count) {
     
           if(isLocationBlock){

			h = dw.getTextAsc() + dw.getTextDesc() + padh * 2+10;

		}else{

			h = dw.getTextAsc() + dw.getTextDesc() + padh * 2;

		}

		if (bg) {


			if(isLocationBlock){

				layoutParams[count].setMargins((int)(x - w / 2 - 10), (int)(y - h / 2 - 10), 0, 0);

				layoutParams[count].height = 90;

				layoutParams[count].width = 90;

				locationMarkerView[count].setLayoutParams(layoutParams[count]);

......... . ....... . ........ ......... . .
}



The `radarText ()` method is responsible for plotting the markers.

# Applying Low Pass Filter to Android Sensor's Readings #

----------


## Overview of Android Sensors ##
The Android sensor framework lets you access many types of sensors. Two very basic types are:
		
1. Hardware Sensors.
2. Software Sensors.

**Hardware sensors** are physical components built into a handset or tablet device. They derive their data by directly measuring specific environmental properties, such as acceleration, geomagnetic field strength, or angular change.<br>
ex. `Sensor.TYPE_ACCELEROMETER, Sensor.TYPE_MAGNETIC_FIELD`

**Software sensors** are not physical devices, although they mimic hardware-based sensors. Software-based sensors derive their data from one or more of the hardware-based sensors and are sometimes called virtual sensors or synthetic sensors.
ex.`Sensor.TYPE_ORIENTATION, Sensor.TYPE_ROTATION_VECTOR`

## Best Practices for Accessing and Using Sensors ##
1. Unregister sensor listeners.
2. Don't test your code on the emulator.
3. Don't block the onSensorChanged() method.
4. Avoid using deprecated methods or sensor types.
5. Verify sensors before you use them.
6. Choose sensor delays carefully.
7. **Filter the values received in `onSensorChanged()`. Allow only those that are needed.**

After we register the Sensors, the sensor reading get notified in `SensorEventListener`'s `onSensorChanged()` method. However, the rate of change in sensor values is such high that if we map these small changes a.k.a 'Noise' the values jump within a large range of values.<br>

<br>We can also specify the `SensorManager`'s delay properties from one of these:<br>
1. `SENSOR_DELAY_FASTEST`<br>
2. `SENSOR_DELAY_GAME`<br>
3. `SENSOR_DELAY_UI`<br>
4. `SENSOR_DELAY_NORMAL`<br>

<br>This is however only a hint to the system. Events may be received faster or slower than the specified rate. Usually events are received faster.
<br><br>Moral of the story is...
> Allow only those values which are useful and discard the unnecessary noise.

The solution for this is to apply a **[Low-Pass Filter](http://en.wikipedia.org/wiki/Low-pass_filter)** on these values.

## A Small Glimpse of Low Pass Filter ##
A [low-pass filter](http://en.wikipedia.org/wiki/Low-pass_filter) is a filter that passes low-frequency signals/values and attenuates (reduces the amplitude of) signals/values with frequencies higher than the cutoff frequency.<br>
Let us take an example of simple signal with values ranging from 0 to 1. The values can be anything between 0 to 1.
Due to some external source, environmental factors such as jerks, vibrations., a considerable amount of noise is added to these signals. These high frequency signals (noise) cause the readings to hop between considerable high and low values.
<br>
<br>
**Enough of Physics...**<br>
Lets understand this in programming concept rather. :)
## Programmatically Apply Low Pass Filter ##
Device's sensor readings add noise data due to high sensitivity of its hardware to various factors as mentioned above. For gaming purpose these highly sensitive values are much a boon. But for application where there has to be smoothness in the readings, these hopping values are a mess.<br>
Lets take an example of [Augmented Reality View](https://github.com/Bhide/AugmentedRealityView.git), where we have to point markers on `Camera` `SurfaceView`.<br>
The high sensitivity causes the markers the change positions very randomly due to the noise.<br>
A Low-Pass Filter concept comes to rescue. We can omit those high frequencies in the input signal applying some suitable threshold and apply the filter output reading to plot the markers.<br>
With this implementation the markers won't hop randomly because we have removed the unwanted high reading values.

Here is the algorithm implementation:<br>
`for i from 1 to n`<br>
`y[i] := y[i-1] + α * (x[i] - y[i-1])`<br>
<br>
Here, α is the cut-off/threshold.<br>
<br>
Lets see how to implement it in Android.<br>
`lowPass(float[] input, float[] output)`<br>
The above method filters the input values and applies LPF and outputs the filtered signals<br><br>
`static final float ALPHA = 0.25f; // if ALPHA = 1 OR 0, no filter applies.`

<br>![image](https://dl.dropboxusercontent.com/u/2906868/low%20pass%20filter.png)
<br>Low-Pass Filter is finally applied to sensor values in `onSensorChanged(SensorEvent event)` as follows:<br><br>
![image](https://dl.dropboxusercontent.com/u/2906868/LPF.png)

<br>An example of this can be found [here](https://github.com/Bhide/AugmentedRealityView.git).
<br>Here i have applied low pass filter for `Sensor.TYPE_ACCELEROMETER` and `Sensor.TYPE_MAGNETIC_FIELD`.
