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
[image1]: ./output_images/image_example.png
[image2]: ./output_images/hog_car_YCrCb_color_space.png
[image3]: ./output_images/hog_notcar_YCrCb_color_space.png
[image4]: ./output_images/img_apply_roi_mask.png
[image5]: ./output_images/lane_w_car_locations.png
[image6]: ./output_images/lane_w_car_locations_another.png
[video1]: ./project_video_output.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.   

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the first code cell of the IPython notebook (function  `get_hog_features()`).  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Below is an example using the `YCrCb` color space with Hue channel and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`. I used `YCrCb` color space because it appears that the difference in the HOG map is most obvious. 


![alt text][image2]
![alt text][image3]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and track the accuracies on 100 test samples. With below parameters, I was able to get 0.9899 accuracy. 
cspace = 'YCrCb'
hog_channel = 'ALL'
spatial_size = (32,32)
histbin = 32
orient = 9
pix_per_cell = 8
cell_per_block = 2
 
#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using the concatenated features (HOG feature, binned color features, and color histogram features). I random splitted the data into training data and test data, and tracked the accuracy on the test data. Code is in the 4th cell of jupyter notebook. 

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I used the following parameters for the sliding window search. The window size is determined by the number of pixel per cell in the original sampling in the. I used 1 cell per step during sliding to avoid false negatives.
```
    window = 64
    nblocks_per_window = (window // pix_per_cell) - cell_per_block + 1
    cells_per_step = 2  
    nxsteps = (nxblocks - nblocks_per_window) // cells_per_step + 1
    nysteps = (nyblocks - nblocks_per_window) // cells_per_step + 1
```

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using HSV 3-channel HOG features plus spatially binned color and histograms of color in the feature vector. To enhance the performance of the classifier, I applied an ROI masked based on the left locations in the image. So the search for vehicle is only within the ROI. The locations of the left lane are detected using the code from last project. 
An example is as below: 
![alt text][image4]

Here are some results on the test images:
![alt text][image5]
![alt text][image6]


---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_output.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.
To reduce false positives, I applied heat map method suggested in the course. Also I applied an ROI masked based on the left locations in the image. The locations of the left lane are detected using the code from last project. 
I also kept a history of heat maps obtained from previous frames and applied thresholding on the averaged heat map. In this way, the results are smoothed. The disadvatange is that the performance get worse because it needs to iterate through the kept history of the heat maps.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The heatmap threshold and the a lot of parameters in the svc trainer and predictor are hard-coded to this problem setting (lighting condition etc.). In other weather condition or lighting condition, these parameters might let the classifier fail. To make it robust, we should try the classifier in different conditions and see whether we need to set these parameters differently given the weather/lighting condition.  

