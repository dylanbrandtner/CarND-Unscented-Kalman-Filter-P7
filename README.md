# Sensor Fusion: Unscented Kalman Filter Project

[//]: # (Image References)
[image1]: ./doc/Final.png  "Final"
[image2]: ./doc/KalmanFilterAlgo.png  "kalman"
[image3]: ./doc/UKF_Roadmap.JPG  "UKF"
[image4]: ./doc/NIS_Charts.png  "NIS"

The overall goal of this project was to utilize an unscented kalman filter to estimate the state of a moving object of interest with noisy lidar and radar measurements. 

The kalman filter is run against a simulator to predict the motion of a car.  The simulator can be found [here](https://github.com/udacity/self-driving-car-sim/releases/). Lidar measurements are red circles and radar measurements are blue circles with an arrow pointing in the direction of the observed angle.  The path of the car is predetermined by the input data set, and the green triangles are the predicted path from the kalman filter.  Here is a snapshot of my final result:

![alt text][image1]

## [Rubric Points](https://review.udacity.com/#!/rubrics/783/view)
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

### Compiling


The main program can be built and run by doing the following from the project top directory.

1. mkdir build && cd build
2. cmake .. && make
3. ./UnscentedKF

### Accuracy

The rubric specifies the maximum RMSE values for each measurement. My final RSME values vs. the limits were as follows:

| Measurement | Max RSME | Measured RSME  |
|:-----------:|:--------:|:--------------:|
|     px      |   0.09   |     0.0852     |
|     py      |   0.10   |     0.0859     |
|     vx      |   0.40   |     0.3166     |
|     vy      |   0.30   |     0.2215     |

Here's a [link to a video recording of my final result](./doc/project_recording.mp4).

### Follows the Correct Algorithm
#### Sensor Fusion algorithm follows the general processing flow as taught in the preceding lessons

The Kalman filter I implemented generally follows the flow described in the lesson materials:

![alt text][image2]

With an unscented kalman filter, the predict and update steps are bit more complex, as described in the diagram below:

![alt text][image3]

#### Kalman Filter algorithm handles the first measurements appropriately

The 'UKF' class initializes all necessary matrices and variables.  Input measurements enter the 'ProcessMeasurement()' function in the 'UKF' class.  The measurements can be either Lidar or Radar data. On the first measurement, states are initialized based on the provided data.  For Radar data, the input polar coordinates are converted to Cartesian coordinates using basic trigonometry and stored here.  

#### Kalman Filter algorithm first predicts then updates

On subsequent measurements, prediction is preformed using the 'Prediction()' function of the 'UKF' class.  First, augmented sigma points are generated using the 'GenerateAugmentedSigmaPoints()' function, and then those sigma points are used to predict new sigma points in the 'SigmaPointPrediction()' function.  This information is then used to predict a new mean state and covariance matrix . 

Next, the Update step is preformed using either the 'UpdateLidar()' or  'UpdateRadar()' functions of the 'UKF' class, depending on the type of input measurement. In either case, sigma points are transformed into the measurement space, a noise covariance matrix is defined, and then 'UpdateHelper()' predicts a new measurement, and sets up innovation, cross correlation, and kalman gain matrices.  Then, predicted measurements are compared against the received measurements, and  this is used to update the mean state and covariance matrix.  Additionally, the NIS (Normalized Innovation Squared) value is updated for the given measurement.  

#### Kalman Filter can handle radar and lidar measurements

Lidar and Radar measurements are handled similarly during the update step.  The primary difference is when setting up the matrix to transform the predicted sigma points into the measurement space.  For Lidar, we simply extract px and py from the sigma points.  For Radar, we extract px, py, velocity, and yaw, and then we transform this information into range, bearing and velocity using trigonometry to match our measurement model.  Also, for Radar, we make sure to normalize all angle values to be between -π  and π throughout the update process.    

### Code Efficiency
Algorithm does not sacrifice comprehension, stability, robustness or security for speed, while still maintaining good practice with respect to calculations. I also avoided code duplication where possible.  I added an 'UpdateHelper() function in the 'UKF' class to avoid duplicate update calculation code.

## Discussion
Again, this project was not particularly work intensive since most of the code was available in the previous lesson materials.  The only real complexity involved was updating the matrices to transform the sigma points to the measurement space for Lidar.  The rest of the Radar update code was portable enough to be reused by just removing the "angle normalization" pieces.     

The most interesting part of this project came with tuning the acceleration noise parameters.   The initial values were obviously much too high, but I wanted to use the NIS calculation to try and understand why, as the course suggested.  I calculated the NIS value for Radar and Lidar and printed them out on each update step.  I tuned the noise values a bit until the NIS started to drop off, and then I started dumping them to a CSV file so I could graph them and their associated 95% line.  In the end, using a longitudinal acceleration noise of 3 m/s^2 and a yaw acceleration noise of .5*π rad/s^2, I was able to achieve the following NIS plots:

![alt text][image4]

The measurements are plotted in blue, and the 95% line is drawn in red.  For Lidar, with 2 degrees of freedom, I drew the 95% line at about 5.9.  For Radar, with 3 degrees of freedom, I drew the 95% line at about 7.8. As you can see, in both plots, it appears only about 5% of the measurements are above the red line.  This, along with my favorable RSME values, gave me confidence I had chosen good noise values to create a consistent unscented kalman filter.   
