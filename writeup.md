## Writeup Template
### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/car_noncar.png
[image2]: ./output_images/rgb_YCr_hog.png
[image3]: ./output_images/sliding_windows.png
[image4]: ./output_images/label_heat.png
[image5]: ./output_images/bboxes.png
[image7]: ./examples/output_bboxes.png
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the class `Feature` of the IPython notebook. It contains all functions related to get features.

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=10`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image2]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters, it was guess and try. Final accuracy was varying between 0.97-0.99. The result is affected by amount of images included for training. It's not easy mixture of parameters.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

Code related to training of linear SVM is at class `Trainer`. It was important to get all features in required shape, one big array of features per single image. I decided to have 0.33 share of testing set. 

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

Code related to sliding windows is at class `Detector`. Finding best parameters of sliding windows was one of the boggest challenges of the project. Problem was not only to find car on the image, it is required to find all rectangles in reasonable time. If the overlap was big, result of detection was great but it took few minutes per single image.
There are 3 main parameters:
* size of the window
I decided to have multiple sizes of the sliding window (128, 96 ,64). Smaller windows were great for detection but it slows down the process and tends to find false positives.

* size of the overlap
It's quite simple, big overlap, great results but slow progress and small owerlap, poor results but fast progress. I had to fin some compromise.

* borders of the area
The simplest decision, limiting width of the image does not bring big improvement, height of the area was cut by half because top half contains trees and sky.

Combination of window size and overlap size with respect to execution time was big challenge for me. I was forced to come with simple dynamic of the parameter. Default size of overlap is equal to half of the window size and when it finds positive result, it grows size of overlap to the size of the window - 2. Evere nex false result grows overlap to original value.


![alt text][image3]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector. Result is compromise between big execution time and great result and poor but fast result. Here are some example images:

![alt text][image4]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./output_video/project_video.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

Save coordinates of all rectangles where detection was positive. From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Example of rectangles and corresponding heatmaps:

![alt text][image4]


### Here the resulting bounding boxes are drawn onto the last frame in the series:
![alt text][image5]



---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Biggest issue was combine all parameters to make it work. It requires a lot of tries and observing which combination gives the best direction to the result.

The pipeline has some minor fails even in current solution. It's seen on video, the car is lost and than is found again. Maybe some improved color preprocessing or kind of edge detection could work better.

I had idea to improve and speed up solution. If large sliding windows define larger areas where objects should be present. Smaller sliding windows use this info and focus on "hot" areas only and refine result. It skips big part of image.

