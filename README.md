### Vehicle Detection Project

This is my solution to Udacity's self driving car term 1 project 5.

Refer to https://github.com/udacity/CarND-Vehicle-Detection for more information.


The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

### Running the code
To run the code, grab the training data from 
https://github.com/udacity/CarND-Vehicle-Detection and step through the vehicle_detection.ipynb.


[//]: # (Image References)
[image1]: ./output_images/car_not_car.png
[image2]: ./output_images/HOG_example.png
[image4]: ./output_images/sliding_window.png
[image7]: ./output_images/output_bboxes.png

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

###Histogram of Oriented Gradients (HOG)

####1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the third and fifth code cell of the IPython notebook.

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![Example images][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![Hog visualization][image2]

####2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and uses both visualization and performance of the
final classifier on validation set to decide on the final parameters.

The final choice use all 3 channels of YCrCb color space, orient=9, pix_per_cell=8, cell_per_block=2.

####3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM at the 8th code cell. I also used spatial features of size (16, 16) and
color histogram features of 16 bins.

The final performance on the validation accuracy is 0.9901 and it took about 26.27 seconds to train
the model.

###Sliding Window Search

####1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

The code is at 9th and 10th code cell of the notebook.

I iterated through the hyperparameter settings by using performance on the test images.
I decided to search only the bottom half of the image to cut the computation and false positive
since the camera position is fixed on the car. I used two different sliding window size of
64x64 and 128x128 to be able to detect car of varying sizes.


####2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned
color and histograms of color in the feature vector, which provided a nice
result.  I also used heatmap to combine the output from the two different sliding window
scales to remove false positive for each frame. Here are some example images:

![Sliding window output][image7]
---

### Video Implementation

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./output_video.mp4)


####2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I did this at the 12th code cell.

I recorded the positions of positive detections in each frame of the video, keeping results
from the last 7 frames.  From the positive detections I created a heatmap and then
thresholded that map (to at least have 4 frames with the boxes) to identify vehicle positions.  I then used
`scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.
I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

If we can spend more time, there are a couple of things to improve the pipeline.

Multi-scale. Right now, I only used 2 different scales but this cause problem when the car enters
the scene and when the car is very far from the view. Adding more scales will help but this cause the pipeline
to be a lot slower. One way to make it faster is to use different scale at different location of the image
(eg. bigger scale when it is at the bottom of the image).

Close cars. Right now, when two cars are very close to each other, the pipeline only detect one bounding box.
This is not ideal, and we should consider how to split the two boxes when computing the heat map and using
the labels function. This might
involve using different threshold and different # of frames.
