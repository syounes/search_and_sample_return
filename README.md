# search_and_sample_return
Robotics Nanodegree Program Project Submission

I'll go through the basic implementation and the necessary steps for the project submission.

First I downloaded the simulator and recorded some data then I got throught the environment setup with the python starterkit and finally I run the Rover_Project_Test_Notebook.ipynb from the project repository folder under the RoboND environmnet and Jupyter Notebook.

Notebook Analysis

   1. Obstacle and rock sample identification.
      In the perspetive transform image we mapped the navigable terrain, the obstacles that appear in dark and we identified the field of vew of camera by creating a mask
      
      ```
      # Define calibration box in source (actual) and destination (desired) coordinates
      # These source and destination points are defined to warp the image
      # to a grid where each 10x10 pixel square represents 1 square meter
      # The destination box will be 2*dst_size on each side
      def perspect_transform(img, src, dst):
           
         M = cv2.getPerspectiveTransform(src, dst)
         warped = cv2.warpPerspective(img, M, (img.shape[1], img.shape[0]))# keep same size as input image
         mask = cv2.warpPerspective(np.ones_like(img[:,:,0]), M, (img.shape[1], img.shape[0]))
         
         return warped, mask
         
      # Define calibration box in source (actual) and destination (desired) coordinates
      # These source and destination points are defined to warp the image
      # to a grid where each 10x10 pixel square represents 1 square meter
      # The destination box will be 2*dst_size on each side
         dst_size = 5 
         
      # Set a bottom offset to account for the fact that the bottom of the image 
      # is not the position of the rover but a bit in front of it
      # this is just a rough guess, feel free to change it!
      bottom_offset = 6
      source = np.float32([[14, 140], [301 ,140],[200, 96], [118, 96]])
      destination = np.float32([[image.shape[1]/2 - dst_size, image.shape[0] - bottom_offset],
                  [image.shape[1]/2 + dst_size, image.shape[0] - bottom_offset],
                  [image.shape[1]/2 + dst_size, image.shape[0] - 2*dst_size - bottom_offset], 
                  [image.shape[1]/2 - dst_size, image.shape[0] - 2*dst_size - bottom_offset],
                  ])
      warped, mask = perspect_transform(grid_img, source, destination)
      fig = plt.figure(figsize=(12,3))
      plt.subplot(121)
      plt.imshow(warped)
      plt.subplot(122)
      plt.imshow(mask, cmap='gray')
     ```
     
   2. ```process_image()``` analysis & worldmap creating

Autonomous Navigation and Mapping 

   1. ```perception_step()``` and ```decision_step()``` explanation and why these functions were modified as they were.
   
   2. Autonomousmode navigation results explanation and how we might improve them.

