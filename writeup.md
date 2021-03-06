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
[image1]: ./output_images/OriginalImage.png
[image2]: ./output_images/YCrCb.png
[image3]: ./output_images/SpatialBinning.png
[image4]: ./output_images/ColorHisto.png
[image5]: ./output_images/HOG.png
[image6]: ./output_images/Combined.png
[image7]: ./output_images/test4.png
[image8]: ./output_images/SlidingWindows.png
[image9]: ./output_images/DetectedWindows.png
[image10]: ./output_images/Heat.png
[image11]: ./output_images/Output.png
[image12]: ./output_images/Labels.png
[video1]: ./project_output_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/dhirendraism/CarND-Vehicle-Detection/blob/master/writeup.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

For feature extraction I tried with multiple different color spaces. The steps that I followed were:
1. Color transform to YCrCb.
2. Spatial Binning.
3. Color Histogram
4. HOG feature extraction.

Code for HOG feature extraction is in Cell 9 of [IPython notebook] (https://github.com/dhirendraism/CarND-Vehicle-Detection/blob/master/writeup.md)

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)`and `cells_per_block=(2, 2)` in all channels of image:

##### Load Image
I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

##### Color Transform
Code for Color Transform extraction is in Cell 4 & 11 of [IPython notebook] (https://github.com/dhirendraism/CarND-Vehicle-Detection/blob/master/writeup.md)
Then I did a color transform to YCrCb:

![alt text][image2]

##### Spatial Binning
Code for Spatial Binning feature extraction is in Cell 5 of [IPython notebook] (https://github.com/dhirendraism/CarND-Vehicle-Detection/blob/master/writeup.md)
![alt text][image3]

##### Color Histogram
Code for Color Histogram feature extraction is in Cell 7 of [IPython notebook] (https://github.com/dhirendraism/CarND-Vehicle-Detection/blob/master/writeup.md)
![alt text][image4]

##### HOG features
Code for HOG feature extraction is in Cell 9 of [IPython notebook] (https://github.com/dhirendraism/CarND-Vehicle-Detection/blob/master/writeup.md)
![alt text][image5]

##### Combined Features
Code for Combined Features is in Cell 11 of [IPython notebook] (https://github.com/dhirendraism/CarND-Vehicle-Detection/blob/master/writeup.md)
![alt text][image6]


#### 2. Explain how you settled on your final choice of HOG parameters.

Here basically I'm using a SVM classifier. I experimented with multiple orientation, pixels_per_cell, cell_per_block. Wiht various combinations my models accuracy was:

Color Space  |  Orient  | pix_per_cell | cell_per_block | accuracy
:-----------:|:--------:|:------------:|:--------------:|:-----------:
YUV | 9 | 8 | 2 | 97%
YUV | 8 | 8 | 2 | 98%
RGB | 9 | 8 | 2 | 97%
RGB | 8 | 8 | 2 | 98%
YCrCb | 9 | 8 | 2 | 99%
YCrCb | 8 | 8 | 2 | 98%
YCrCb | 7 | 8 | 2 | 98%

However, the accuracy was already very high with default parameters and out of the 
box linear SVM so I did not feel compelled to explore that much more. I ended up just using `YCrCb`, `orient=9`, `pix_per_cell=8`, and `cell_per_block=2`. 


#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM by creating a training set from the given vehicle and non vehicle images. Since both datasets had ~9000 images, I did not need to balance the classes. I shuffled the data and split 20% into a validation set and 80% into a training set.

To extract features I used spatial binning, color histograms and HOG features. I concatenated them into a large feature vector and used sklearn.StandardScaler to normalize my data. I fed this data into the svm classifier and used default parameters.

Code can be found in is in Cell 11 of [IPython notebook] (https://github.com/dhirendraism/CarND-Vehicle-Detection/blob/master/writeup.md)

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

The logic for my sliding window search is contained in slide_window and generate_windows method on lines 156-203.

First, I restricted the range of pixels to slide my windows to the bottom half of the image because that is where the road is and where I expect to find images. Initially I slide my 64x64 windows with a 75% overlap. I found that by increasing the overlap from 50% to 75% I was able to find more "hot windows" and made my heatmaps stronger downstream.

However, I found that my bounding boxes were too small for some of my images (like when the car is too close to the camera). So I did the same process but with larger windows (128x128 sliding windows and 160x160 windows sliding windows) to detect these larger subimages.

In short, I used 3 scales (64x64, 128x128, and 160x160 windows)

##### Sliding Window
![alt text][image8]
##### Positive Windows
![alt text][image9]
##### Heatmap
![alt text][image10]
##### Output
![alt text][image11]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

| Sliding Window | Positive Windows | Heatmap | Output |
|:--------------:|:----------------:|:-------:|:------:|
|![alt text][image8]|![alt text][image9]|![alt text][image10]|![alt text][image11]|

---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_output_video.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video. From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions. I then used scipy.ndimage.measurements.label() to identify individual blobs in the heatmap. I then assumed each blob corresponded to a vehicle. I constructed bounding boxes to cover the area of each blob detected.

Here's an example result showing the heatmap from a series of frames of video, the result of scipy.ndimage.measurements.label() and the bounding boxes then overlaid on the last frame of video.

Code can be found in is in Cell 15 of [IPython notebook] (https://github.com/dhirendraism/CarND-Vehicle-Detection/blob/master/writeup.md)

Additionally, to deal with false positives I kept track of heatmaps from previous frames. I maintain a deque of heatmaps from the past 5 frames and add them together to get the "final heatmap" (see `generate_heatmap()` Cell 11 of [IPython notebook] (https://github.com/dhirendraism/CarND-Vehicle-Detection/blob/master/writeup.md)). I then apply a threshold to this cumulative total.

### Here is example of heatmaps:
|Heatmap|
|:------------------:|
|![alt text][image10]|
### Here is the output of `scipy.ndimage.measurements.label()` on the integrated heatmap:
![alt text][image12]

### Here the resulting bounding boxes are drawn onto the last frame in the series:
![alt text][image11]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The speed of performance can definitely be improved. For example, improvements can come from multi-scaled window being called on a single instance of HOG feature generation. The pipeline still occasionally have false-positives. To be more robust we can leverage Advanced-Lane-Finding project and restrict searches in lanes going in the same direction. An increase in sample data, and better resolution would also result in more robust classifiers. This may have been provided in augmented dataset from udacity but wasn't combined into this submission.  

