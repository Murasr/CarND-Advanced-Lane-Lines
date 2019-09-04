## Advanced Lane Finding

The Project
---

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

###Rubric Points

A class called **LaneFinding** is created to store the needed information between the subsequent frames. The camera calibration matrix calculation, perspective transformation matrix information and inverse transformation matrix are calculated at the **__init__** of the class. The method **process_image** takes an image and returns an image with overlayed lane and the curvature, car position calculated.

1) **Camera Calibration:** 
The method **calibrateCamera** calculates the camera matrix and the distortion coefficients for the given chessboard images using cv2.calibrateCamera() method. The undistortion is done by the method **undistortImage** which makes use of camera matrix, the distortion coefficient and the cv2.undistort method. The undistorted camera calibration images are saved under **output_images/camera_cal** folder.

2) **Distortion Correction **
The distortion correction is applied to test images using the **undistortImage** method. The undistorted test images can be found under the **output_images/undistorted_images** folder.

3) **Binary Image Creation**
A fixed kernel size of 3 is used for all binary image calculations. The following are applied as follows:
   a) Sobel x gradient with thresh=(20, 100)
   b) Sobel y gradient with thresh=(40, 100)
   c) Magnitude threshold with mag_thresh=(50, 100)
   d) Direction threshold with thresh=(0.7, 1.3)
   e) Saturation threshold in HLS colorspace with s_thresh=(150, 255)
   
The binary images are created with the below combination:
**combined[((gradx == 1) & (grady == 1)) | ((mag_binary == 1) & (dir_binary == 1)) | ((s_binary == 1))] = 1**
The binary test images can be found under **output_images/binary_images** folder.

4) **Perspective Transform**
The method **calculatePerspectiveTransform** calculates the transformation matrix. The source and destination mapping is done as follows:

src = np.float32([[600, 445], [680, 445], [1060, 679], [248, 679]])
dst = np.float32([[200, 0], [930, 0], [930, 719], [200, 719]])

Kindly note that assymmetric rectangle used in prespective transformation

The perspective transformation is then applied to the test images and the output can be found under **output_images/perspective_transform** folder.

5) **Lane pixel Detection**
After applying the perspective transform, the lane pixels are detected as follows:

a) Take the histogram of the bottom half of the image. Find the peaks in the left and right half of the image

b) For the **first image** fit the polynomial coefficients around the left and right peaks using **sliding window** approach. This is done by the method **fit_polynomial_sliding_window**. The hyperparameters used are as below:
    # HYPERPARAMETERS
    # Choose the number of sliding windows
    nwindows = 9
    # Set the width of the windows +/- margin
    margin = 100
    # Set minimum number of pixels found to recenter window
    minpix = 50
    
c) For the **next images**, search around the region of previous frame polynomial area. This is done by the method **search_around_poly**. The region margin used is 50 pixels. An example image with lane area detected can be found under the folder **output_images/lane_curve_detected_images"

d) The next step is to do sanity check on the obtained left and right polynomials. It is done by the method **doLaneSanityCheck**. The sanity check basically checks for the minimum and maximum pixel distance checks for the pixel line at the bottom of the image and pixel line at the top of the image. 

    # Do some validity checks for bottom distance
    if( (600 > bottomDistanceInPixels.astype('int')) | (bottomDistanceInPixels.astype('int') > 1100)):
        valid = False
        
    # Do some validity checks for top distance
    if( (500 > topDistanceInPixels.astype('int')) | (topDistanceInPixels.astype('int') > 1200)):
        valid = False
        
    # If the difference distance between bottom and top line is more than 400 pixels, then it is invalid
    if(diffDistanceInPixels.astype('int') > 400):
        valid = False



Only if the sanity check is passed, store the polynomial parameters. If not valid, calculate once again using **sliding window** approach.

An example image with lane area detected can be found under the folder **output_images/lane_curve_detected_images"

6) **Lane Curvature Measurement and Position of car**
The method **measureCurvatureReal** calculates the right and left curvature of the lane in meters. The forward lookahead distance is considered to be 50 meters. The curvature is calculated using the radius of curvature formula scaled to metres at 50% of the y_max (ie., at y=360).

The position of car is calculated by the method **findPositionOfCar**. Horizontal distance in pixels is first calculated at y_max and the width of the image is considered as 1130 pixels to account for assymmetric perspective used. The distance is later converted to metres and then position of the car from centre of the image is shown.

7) **Reverse Transformation of Lane**
The lane identified on the warped image is then inverse transformed by the method **doReverseTransform**. Note that different src points are used to find the inverse transformation matrix so that lanes are extended upon the car bonnet.
The final output image can be found at **output_images/final_output_image.JPG**

##PipeLine Video

The above steps are applied to the video "project_video.mp4". The output can be found at **project_video_output.mp4**.

##Discussion
Currently it can be seen at some frames, the lane extends to the guard rails on the left especially during the changes in the brightness of the lane. The above solution can be further improved to do better whenever there is change in brightness of the lane.