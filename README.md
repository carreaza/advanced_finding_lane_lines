Advanced Lane Finding - Carlos Arreaza - Jan 2017

Camera Calibration
Briefly state how you computed the camera matrix and distortion coefficients:
I used all the camera calibration images and ran a for loop to go through them using ret, corners = cv2.findChessboardCorners(gray, (9,6),None). If ret == True means that the function was able to find the 9,6 square intersections. The corners results from findChessboardCorners was appended to the image points , and the same object points (9,6 grid) was appended to object points. cv2.calibrateCamera was used to calibrate the camera using the image points and object points (output is mtx which is the calibration matrix).

Provide an example of a distortion corrected calibration image: See images in jupyter notebook.

Pipeline (test images):
Provide an example of a distortion-corrected image: see jupyter notebook.

Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result:
In the jupyter notebook I try to experiment and see how sobelx, sobely, gradient direction, gradient magnitude, and color threshold can be used to detect lane lines. First, I plot each of these thresholds separately, and by trial and error see what threshold values (min and max) work best for each technique. There is a function that creates a binary threshold image for each of this techniques (abs_sobel_thresh(), mag_threshold(), dir_threshold(), color_threshold()). Then, in the create_binary() function, I use these previously created funcions to 'combine' the thresholds together, and create a single image that hopefully shows the lane lines.

NOTE: sobelx gets lane lines, however sobely catches too much noise. grad magnitude threshold does not work well for different road colors/shades/lightness, grad direction seems to work well to capture the lane lines however also has too much surrounding noise. Finally, the 'S' value in the HLS image seems to capture yellow/white lines.

Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image:
I created funcion warp() to create the perspective transform. The sample images can be found in the notebook just after the warp() function. The src and dst points were manually chosen, however these are implemented with respect to the image size, as there are a couple of images of different sizes. The perspective transform for a straight road show parallel lines, which is what we want.

Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
The algorithm to detect lines is in detect_lanes() where the binary thresholded image is the only input. Using a histogram, and dividing the image by half horizontally, one can determine using np.argmax() where the average or peaks of each line (left and right) are. Then, slicing the image in increments of 90 pixels, and using np.where we can find the indeces of wherever there are 1's in the binary image, inside of an offset (left to the offset and right to the offset). After finding these indeces, we add together the x and y points for each line, left and right, respectively. Finally, using the collection of x and y points we use np.polyfit for a 2 degree line. The lines are then the output of the function.

Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.
The curvature of the lines is calculated in curvature(). With the inputs being the x and y values of the left and right lines respectively, new lines are constructed using np.polyfit(), using the pixel to meters transformation values xm_per_pix and ym_per_pix to get the lines in the units of meters. The curvature is then measured at the maximum value of the y axis (at the bottom of the image) using the radius of a curve formula.

The center of the car is found in function car_center_line() which takes as inputs the two lines and the image. With the lines, we find the x intercept in the bottom of the image, and calculate the center of the lines with respect to the center of the image, which should be the distance off center for the car. xm_per_pix is then used to transform pixel values to meters. 

Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.
The examples of the test images can be found in the jupyter notebook using final_pipeline().

Pipeline (Video)

Provide a link to your final video output. Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!)
See video called project_video_output.mp4.

Discussion
Briefly discuss any problems / issues you faced in your implementation of this project.
It is difficult to reduce the noise as the algorithm is not perfect and some lines are not caught. The tunning of the parameters of the create_binary() function was the hardest part as I would have to try several threshold values and methods to find one that would suit the images.

Where will your pipeline likely fail? 
The pipeline is really good at finding solid yellow lines, and also good at finding solid white lines. However, it performs poorly when there are white segments, and even worse when these segments are noise, small, or even inexisting. The algorithm relies on having some sort of difference between the colors or shapes in the road, thus if the road has no lanes, this algorithm would definitely fail.

What could you do to make it more robust?
I could add a moving average or learning algorithm to store the previous lines, so whenever the algorithm can't find a line, it can always show what was previously found.
