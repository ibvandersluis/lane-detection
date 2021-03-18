# **Finding Lane Lines on the Road** 

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./test_images_output/step1.jpg "Grayscale"
[image2]: ./test_images_output/step2.jpg "Gaussian Blur"
[image3]: ./test_images_output/step3.jpg "Canny Edges"
[image4]: ./test_images_output/step4.jpg "Masked Image"
[image5]: ./test_images_output/step5.jpg "Lane Lines"
[image6]: ./test_images_output/step6.jpg "Final Image"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of the following steps:

1. Using the helper function `grayscale()`, produce a grayscale version of the image.

    ![image1]

2. Using the helper function `gaussian_blur()` and kernel size of 9, apply Gaussian smoothing to the grayscale image.

    ![image2]

3. Using the helper function `canny()`, run canny edge detection on the smoothed image. The low threshold was set to 50 and the high threshold was set to 150.

    ![image3]

4. Get the image shape and define the vertices of a quadrilateral masked area in reference to the image shape. Using the helper function `region_of_interest()` and these vertices, apply a mask to the image resulting from the previous step.

    ![image4]

5. Using the helper function `hough_lines()`, extract the Hough transform lines using the following parameters:
    - rho = 2
    - theta = np.pi/360
    - threshold = 30
    - min_line_len = 30
    - max_line_gap = 40

    ![image5]

6. The `hough_lines()` function then calls the `draw_lines()` helper function to mark the lanes, and the lines are then superimposed over the original image using the `weighted_image()` helper function.

    ![image6]

In order to draw a single line on the left and right lanes, I had to modify the `draw_lines()` function. I did this by collecting information about the right and left lanes before using this information to produce an average line for each side, and draw that line on the image. This was achieved as follows:

1. For each line extracted from `hough_lines()`, use the endpoints to calculate the slope and intercept of the line.
2. If the slope is positive, the line belongs to the left lane. Otherwise, it belongs to the right. Append the slope and intercept to the appropriate lists accordingly.
3. Calculate the averages for each list. The result is an average slope and intercept for the left lane, and the same for the right.
4. Using these averages, calculate the x-values for the endpoints of each lane. I already know the y-values because they are the same as those used for the masked area.
5. Now I have the endpoints for lines that approximate the right and left lanes. I draw them on the image using the `cv2.line()` function.

### 2. Identify potential shortcomings with your current pipeline

One shortcoming is that if any of the frames cannot produce a single line for either the right or the left lane, the algorithm will fail with a divide-by-zero error (you cannot get the average slope from an empty list of slopes).

Another shortcoming is that the pipeline expects lanes to be roughly straight, which causes the lane detection to become confused when the lane is turning to the left or right.

### 3. Suggest possible improvements to your pipeline

A possible improvement would be to store the most recent slope and intercept for both the right and left lanes, and if a frame fails to find a line for the right or the left, the  most recent values are used instead. This allows the program to finish by making an educated guess of the where the lanes are. To prevent this fallback from becoming harmful, a threshold `n` could be set. If a given lane does not have a line detected for it `n` times in a row, the pipeline fails.

Another potential improvement could be to expect non-linear lanes, making the pipeline more robust to curves in the road. This could be done by using splines to approximate the lane boundaries instead of the Hough transform.