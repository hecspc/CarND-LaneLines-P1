# **Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)
[original]: ./test_images/solidYellowCurve2.jpg "original"
[blur]: ./test_images_output/blur_solidYellowCurve2.jpg "blur"
[masked]: ./test_images_output/masked_solidYellowCurve2.jpg "masked"
[lined]: ./test_images_output/lined_solidYellowCurve2.jpg "lined"
[oneline]: ./test_images_output/oneline_solidYellowCurve2.jpg "oneline"
[image1]: ./examples/grayscale.jpg "Grayscale"
[result]: ./test_images_output/solidYellowCurve2.jpg "result"
[challengearea]: ./challengeArea.png "Challenge Area"
[challengelane]: ./challengeLane.png "Challenge lane"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline can be found as a function called `lane_finder`. It consisted of 5 steps. First, I converted the images to grayscale, and then I apply a gaussian filter to blur the image so we can smooth the edges. For that, I have used a kernel size of 5, since it gave me a good result on test images.

![Blur images][blur]

We find the edges on the blurry image using the Canny transform within the thresholds 50 and 150.

Since we are interested on lane and we have a fixed camera, we apply a clipping mask in order to isolate only the lane perspective. This an example result of one the test images

![masked images with edges][masked]

Now we have an image with the lane isolated. We need to find the proper lines that the image contains. In order to achieve this, we transform the image into the Hough space using this parameters:
```
  rho = 2
  theta =  np.pi/180
  threshold = 15    
  min_line_length = 35
  max_line_gap = 20    
```
As a result, we have a set of segments corresponding to the edges of lane lines.

![lined images with edges][lined]

We want to find the lane where the car is driving. We need to know the left and the right lines for the lane. In order to find both lines, I have created a function called `average_lines`. This function loops through all the lines array from the Hough space. It is fair to say that all the lines we get corresponds to the lane lines. Due to the perspective, we can distinguish the left line from the right line according to the sign of the `slope < 0 ? left line : right line`.

We have an array of `left_lines` and another of `right_lines`. Now, we want to find the left and right lines. We know already the top and bottom margins for both lines, which corresponds to the top and bottom of our mask. So the only value unknown is the slope.

According to the previous assumption, all the lines in the array `left_lines` belong to the left line of the lane. Since we are only interested on the slope we can average all the points and calculate the slope of the resulting average line. Once we know the slope, we can calculate our `left_line` that will be the intersection with the top and bottom margin of our mask. The result can be seen in the next picture

![lined images with edges][oneline]

Combining this image with the original, we can check that the lines we have found match exactly with our lane.
![result images with edges][result]

I find this pipeline works pretty well with static images and with the first two videos `solidWhiteRight.mp4` and `solidYellowRight.mp4`. But when we try to use the same pipeline with the challenge video, we can see we have some issues trying to find the lines.

![Challenge area][challengearea]

In the area shown in the image above we have some problems trying to find the right line. Actually it is really hard to see with your own eyes.

In order to resolve this issue we make one important assumption: The lane exist and doesn't change abruptly its direction but in a continuous mode. Basically what this means is that if we can not recognize one of the lines in a given moment, we can assume the line follows the same direction.

So what we do is to average the lines on time. We have buffer of left lines `left_lines_history` and and another one for the right lines `right_lines_history`. We take the last 30 lines (around the lane during the last second if the video is 30 fps) and we average the lines.

Just with this, we can see an improvement but still we are having false lines. To discard these false readings we only take those lines whose slope is less that 2 times the standard deviation or 2 sigma.

If we apply this algorithm to the challenge video, the result is not perfect but quite accurate and the most important is that we don't have abrupt changes .

![Challenge lane][challengelane]

### 2. Identify potential shortcomings with your current pipeline

This is a very basic algorithm and we can identify several shortcomings:

* We are relaying on contrast to find the edges. We depend on light conditions, shadows, line colors.
* Cars or other objects, like other road paintings, inside the region of interest can be identified as lines too.
* We are taking the assumption we don't have abrupt changes on the direction of the lane. While this could be correct on highways, on other kind of roads, like city streets or mountain roads, this assumption fails.

### 3. Suggest possible improvements to your pipeline

In this project we are using opencv to find the edges of the lines.
Since the images are taking from a fixed camera in the car and we know the fov of given camera, we can use this knowledge to calculate the distorsion of the perspective of given camera.

This will help us to calculate and to play with distances so we can know more accurately where to find the lines.

We could use a polynomial fit so we can adapt better to more curvy tracks.
