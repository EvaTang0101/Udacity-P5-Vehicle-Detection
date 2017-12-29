# Udacity-P5-Vehicle-Detection
## Step 1: Extract vehicle features
### 1.1 Import vehicles and non vehicles data
The GTI vehicle data were extracted from videos, and the images are time series. In order to avoid getting similar images in the training and test set, I have only used 1/5 of the GTI vehicle images. I did the same thing for the GTI non-vehicle data so that there are equivalent level of samples for both vehicle and non-vehicle categories.

### 1.2 Define helper functions to detect vehicle features
Here I defined a helper function to extract hog features, spatial features and histogram features.

### 1.3 Define feature extraction parameters
- color_space: YUV and YCrCb are both doing a good job in detecting vehicles. HSV, LUV, although performing equally well on svc accuracy, often misclassify cars on the test images. More specifically, the white car on the right edge of the image is often only partially recognized.

- orient: number of orientation indicates the number of directions to be captured in hog features. I have also tried different orient numbers. The higher the orient, the higher the prediction accuracy, but the longer the training time. And when orient gets higher than 9, the improvements in accuracy is not obvious. An orient of 9 has a good balance of accuracy and speed.

- pix_per_cell: including more pixels in a cell means we are grouping more pixels for averaging out directions. Higher pixel per cell speeds up calculations but also loses accuracy. A pix_per_cell of 8 gives a good trade-off between speed and accuracy.

- cell_per_block: number of cells to normalise over. The higher the number, the longer the feature vector. Using a cell_per_block of 4 somehow introduced more false positive. I selected a cell_per_block of 2 for this practice.

- hog_channel: I included the hog features of all channels. It gives a better detection result compared to using a single channel.

- spatial_size: the more granular the histogram bins are, the higher the prediction accuracy, but also the longer the training time. I used 32 as the binning dimension.

### 1.4 Extract car features and non-car features for sample images
Here I have extracted hog features, spatial features and histogram features for car and non-car sample images. The sample feature images below show the difference in colors and hog directions between car and non-car images.

### 1.5 Extract car features and non-car features for all car and non-car data

### 1.6 Build a classifier to detect car and non car images
I first normalised the feature vectors so that different features can be compared on a simlar scale. Then the car and non-car data are split into training and test set, and a SVM and a decision tree classfier is trained. Both models produce quite good accuracy results, but SVM is much faster in training and making predictions.

## Step 2: Scan through images, and find windows with cars detected
### 2.1 Define helper functions to detect windows
I have implemented both the find_car and sliding window methods to detect hot windows with car images. The advantage for using find_car function is that it only grabs hog features once, while using sliding window, we grab hog features in a loop at each sliding window. I ended up using find_car function.

The find_car algorithm scans through the images using sliding boxes. For each box, I ran the classification model to predict if it contains a car or not. For the ones predicted to have a car, the box coordinates are recorded.

### 2.2 Define heatmap functions to select hot windows
The following heatmap functions help reduce false positive boxes. Heatmaps are drawn on images with detected windows, and only areas that are boxed for over certain amount of times are selected as "hot windows".

Initially, I noticed the boxes do not completely cover the cars, because the hot window threshold is filtering out the boxed areas with little overlapping boxes. Therefore, I have boosted the heat level by 2 for areas overlapping areas. This way, the car edges are included in the final hot windows.

### 2.3 Detect cars on test images
I have applied the car detection function on the lower half of the image where the roads run. I used different scaling factors accross the image, since the cars in the middle of the image are supposed to be further away and smaller, and therefore require smaller bounding boxes, while the cars near the bottom of the image are suppossed to be closer and bigger, and thus require bigger bounding boxes. 

I have applied a threshold of 1 on the heatmap. Areas without overlapping bounding boxes will be discarded. The areas remaining are those with higher confidence, and this method helps reduce false positive predictions.

### 2.4 Define process pipeline to detect cars in video frames
I have kept track of past heatmaps in previous frames, and use the average as the current heatmap. This process allows the model to:
1. stablize heatmap or hot windows across each frame
2. reduce false positive, since incorrectly detected boxes in the previous frames may not re-appear in the next frames. Averaging reduces the heat of those false positive boxes.

### 2.5 Apply process pipeline on videos

## Potential Improvements
1. improve algoritm to include bounding boxes around cars further away at the end of the road
2. apply bounding boxes on the cars on the opposite traffic. It can be very helpful for self driving cars when there are no barrier in between two way roads
3. better train the model to reduce false positive
4. accelerate the process pipeline by using more efficient features vectors and algorithms
5. improve the bounding boxes to cover the full car, and bound the car tighter
6. when cars just enter into the camera coverage, the current model could not quickly capture the car
7. stabolize the bounding box across frames
