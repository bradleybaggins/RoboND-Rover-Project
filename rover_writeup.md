
## Project: Search and Sample Return
### Writeup Template: You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---


**The goals / steps of this project are the following:**  

**Training / Calibration**  

* Download the simulator and take data in "Training Mode"
* Test out the functions in the Jupyter Notebook provided
* Add functions to detect obstacles and samples of interest (golden rocks)
* Fill in the `process_image()` function with the appropriate image processing steps (perspective transform, color threshold etc.) to get from raw images to a map.  The `output_image` you create in this step should demonstrate that your mapping pipeline works.
* Use `moviepy` to process the images in your saved dataset with the `process_image()` function.  Include the video you produce as part of your submission.

**Autonomous Navigation / Mapping**

* Fill in the `perception_step()` function within the `perception.py` script with the appropriate image processing functions to create a map and update `Rover()` data (similar to what you did with `process_image()` in the notebook).
* Fill in the `decision_step()` function within the `decision.py` script with conditional statements that take into consideration the outputs of the `perception_step()` in deciding how to issue throttle, brake and steering commands.
* Iterate on your perception and decision function until your rover does a reasonable (need to define metric) job of navigating and mapping.  

[//]: # (Image References)

[image1]: ./misc/rover_image.jpg
[image2]: ./calibration_images/example_grid1.jpg
[image3]: ./calibration_images/example_rock1.jpg

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded).
#### Add/modify functions to allow for color selection of obstacles and rock samples.
The provided sample functions in the Test notebook provided the following output:

Testing the provided base functions on newly captured sample data yielded the following output:

![alt text][image1]


#### 1. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result.

By applying the perspective transform and processing the rover image with the default color threshold, the rover can begin to identify pixels which represent navigable terrain. By combining those pixel positions with the rover’s telemetry, the rover can also build the 'worldmap' using the 'pix_to_world()' function. Any pixels which pass the RGB threshold are deemed navigable and assigned to the 3rd channel of the 'worldmap'. Obstacles will be assigned to the first channel of the 'worldmap' for any pixels which do not pass the color threshold. Rocks will need to pass a different threshold targeting a more specific color.

![alt text][image2]


### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.

#explanation of how and why these functions were modified as they were.

In order to identify navigable terrain for the rover’s decision script to drive through, I combined the preparatory image-processing functions in the 'perception_step()' formula to take advantage of the Rover class. Following the steps which match the Test Notebook, I first input the source and destination fields used in the lessons. Since the environment is the same, the same inputs should work for the perspective transform.

I then apply the 'perspect_transform()' function using the Rover’s current image, applying the transform according to the source and destination of the environment and assigning the output to 'threshed'. Applying the color thresholding to the image returns 1s when the pixel is deemed navigable by passing the RGB threshold, so the opposite of that area would be deemed an obstacle. In order to maximize the fidelity of the rover’s mapping, I added a mask to the 'perspect_transform()' function output to only cover pixels which the rover can actually see. This way, the rover only deems terrain it sees but which does not pass the color threshold as impassible. Returning 1s when threshed = 0 and 0s when threshed = 1 for only pixels included in the visible set of pixels ( * mask) returns the observable set of obstacle pixels, assigned to 'obs_map'.
The rover’s 'vision_image' channels are then assigned values from these thresholded sets of pixels, given the brightest value in the first channel for obstacles and the brightest value in the 3rd channel for navigable terrain.

The image coordinates are converted to coordinates in the rover’s perspective using the 'rover_coords()' function included in the notebook. These adjusted sets are then each fed to the 'pix_to_world()' so they can be rotated and translated and fed to the 'Rover.worldmap'. Navigable areas are assigned to the 3rd channel of the 'Rover.worldmap' with positive observations receiving a value +10 each time they are observed and obstacles are assigned to the 1st channel +1 each time they are observed. Doing this maps the environment as the camera is fed new images that depict the environment.

Turning those navigable pixels into polar coordinates to send the 'decision_step()' a nav angle will inform 'decision_step()' how much to add or reduce to the 'Rover.yaw'. The 'find_rocks()' function was added to apply a new threshold to the image, but also assign the output to a distinct color channel. The output 'rock_map' is similar to the 'threshed' or 'obs_map' sets and is then processed in the same, first to coordinates in the rover’s perspective via 'rover_coords()', then to the world coordinates using 'pix_to_world()', and finally to a set of telemetry instructions via 'to_polar_coords()'. The closest point of the rock is assigned to 'Rover.worldmap' in the 2nd color channel. The 'vision_image' is also updated in a similar way showing pixels which pass this other threshold.

#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

All this information is enough to provide the rover with nav angles that enable it to drive autonomously. In initial runs of the rover in autonomous mode, I had noticed that the rover succeeded in identifying several rocks within the first 100-150 seconds, but began to struggle with map fidelity. I adjusted the 'throttle_set' in the 'RoverState()' down from 0.2 to 0.15 to map more slowly, but hopefully achieve a greater degree of accuracy in mapping. I also would reduce the 'Rover.steer' parameter to hopefully achieve greater mapping fidelity when the rover is turning and the perspective warping is inherently less accurate.

![alt text][image3]
