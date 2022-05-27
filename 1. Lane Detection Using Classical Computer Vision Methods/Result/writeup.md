# **Finding Lane Lines on the Road** 

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps. 

Firstly, I converted the images to grayscale, then I applied a Gaussian noise kernel of 5 on the graycale image. 

Secondly, the image is ready for the Canny edge detection to from the all the edges on the image with respect to the lower and upper thresholds. The lower and upper thresholds I chose is 50 and 100 respectively. The reason for that is this two thresholds can identify the edges for grey-ish white lanes very well. 

Thirdly, after applying the Canny edge detection on the Gaussian blur image, the next step is to find the region of interest on the Canny edge image. The ploygen I used for the mask frame is a trapezoid shape that starts from bottom left conner to the top left end of the white lane and continues to the top right end of and finished at the bottom right conner. This trapezoid will cover most car lanes in the test images and videos. 

Fourthly, after finding the region of interest, I applied hough_lines to the cropped Canny image. I choose 50 as the min line length to be enough to fully capture small lane segment and the filter out the noisy litte white dot on the lanes. Same thing for the max_line_gap. 

Finally, with the red rectangle drawn with hough lines around the white lane, apply the weighted image to get the origin image back. At this point, I have disjointed red line for the lane and this means I need to modify the draw_lines() function to extrapolate the car lane.

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by find the slope for each individual line from all the hough lines first. The equation for slope is (y2-y1)/(x2-x1). All the slope will be checked to see if they are too gradual. If the absolute value of that slope is smaller than 4.5, it will be consider as a horizontial line and ignored. 

Secondly, I separete those hough lines based on which side of the car lane the line is placed at. Since the axis is inverted, if the slope is negative, the line is goind upward which means it is a left side lane and vice versa. With the grouped hough lines, I find the average of all the left and right slopes and y-intercept by polyfiting any 2 points from the hough line groups. Once I have the average y-intercept and slope, I reverse the y = kx+b equation in order to find x coordinates from the suitable y coordinates. This will produce a extrapolated and extended line to identify both side of the car lane. 

One problem arised from this implementation is that due to the extrapolation from multiple line segement, the slope and y-intercept value might fluctuate suddenly and result the lane lines in the video shakes. 

The method I used to mitigate that shake is by introducing a averging factor for slope and y-intercept that comes from the previous frame of the video. Everytime when calculate the slope and y-intercept, I add up the current slope and y-intercept with the value from the previous frame and divide the sum by 2 to get the average slope and y-intercept. This will reduce the suddenly change and make the line changes smoother.


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be what would happen when there is a traffic jam on the highway. Car will stuck on the highway one by one with a very little space between each. This will block the view of the car lanes on the road. Which will cause this road line detector failed to find line in front of the car.

Another shortcoming could be the shaking lane indicator. As I mentioned above, my lane lines in the video shakes. I have already reduce the shake a lot by implement the averaging factors. However, the some smaller jumps still exists. This will be another shortcoming of this road line detector.


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to not only detector the line on the car lane, but also detect the lines of the previous car. If there is no car lane avaliable to perform the lane line detection, use the car in front to estimate the lane lane. If the prevous car appears as a rectangle shape, it means the lane is going straigth. If the car appear to have slanted lines and toward left in its shape, then the lane is going left and right if slanted right. This will fix the first potential shortcoming.

Another potential improvement could be to increase the lookup window size for the averaging factor of lope and y-intercept. Right now I only lookup the previous 1 frame. With the increasing lookup window size, we can look back 2 or 3 frames which will result a better averaging and further reduce the line jump.
