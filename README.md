# search_and_sample_return
Robotics Nanodegree Program Project Submission

I'll go through the basic implementation and the necessary steps for the project submission.

I got throught the environment setup with the python starterkit and finally I run the Rover_Project_Test_Notebook.ipynb from the project repository folder under the RoboND environmnet and Jupyter Notebook.

Notebook Analysis

   1. Obstacle and rock sample identification.
   
      Perspective Transform
      
      ![Alt text](/misc/example_grid1.jpg?raw=true "Title")
      
      I've used the example grid image above to choose source points for the
      grid cell in front of the rover (each grid cell is 1 square meter in the sim)
     
      ```
      def perspect_transform(img, src, dst):
           
         M = cv2.getPerspectiveTransform(src, dst)
         warped = cv2.warpPerspective(img, M, (img.shape[1], img.shape[0]))# keep same size as input image
         mask = cv2.warpPerspective(np.ones_like(img[:,:,0]), M, (img.shape[1], img.shape[0]))
         # Return the warped image and his mask
         return warped, mask
      ```
      
      These source and destination points are defined to warp the image
      to a grid where each 10x10 pixel square represents 1 square meter
      The destination box will be 2*dst_size on each side
      
      ```dst_size = 5```
      
      Setting the bottom offset
      
      ```bottom_offset = 6```
      
      ```
      source = np.float32([[14, 140], [301 ,140],[200, 96], [118, 96]])
      destination = np.float32([[image.shape[1]/2 - dst_size, image.shape[0] - bottom_offset],
                        [image.shape[1]/2 + dst_size, image.shape[0] - bottom_offset],
                        [image.shape[1]/2 + dst_size, image.shape[0] - 2*dst_size - bottom_offset], 
                        [image.shape[1]/2 - dst_size, image.shape[0] - 2*dst_size - bottom_offset],
                        ])
                        
      warped, mask = perspect_transform(grid_img, source, destination)
      ```
      
      ![Alt text](/misc/perspective.png?raw=true "Title")

      Color Thresholding
      
      For obstacles we can just invert the color selection that we used to detect ground pixels. 
      Everything above the threshold is navigable terrain, then everthing below the threshold must be an obstacle.
      
      Threshold of RGB > 160 does a nice job of identifying ground pixels only
      ```
      def color_thresh(img, rgb_thresh=(160, 160, 160)):
         # Create an array of zeros same xy size as img, but single channel
         color_select = np.zeros_like(img[:,:,0])
         # Require that each pixel be above all three threshold values in RGB
         # above_thresh will now contain a boolean array with "True"
         # where threshold was met
         above_thresh = (img[:,:,0] > rgb_thresh[0]) \
                     & (img[:,:,1] > rgb_thresh[1]) \
                     & (img[:,:,2] > rgb_thresh[2])
         # Index the array of zeros with the boolean array and set to 1
         color_select[above_thresh] = 1
         # Return the binary image
         return color_select  
         
         threshed = color_thresh(warped)
         obs_map = np.absolute(np.float32(threshed) - 1) * mask
      ```
      ![Alt text](/misc/threshold.png?raw=true "Title")
      
      For rocks, we think about imposing a lower and upper boundary in our color selection to be more specific about 
      choosing colors.
      
      RGB of 110,110 and 50 can detect the yellow color of the rocks within the images
      
      ```
      def find_rocks(img, levels=(110,110,50)):
         rockpix = (img[:,:,0] > levels[0]) \
                 & (img[:,:,1] > levels[1]) \
                 & (img[:,:,2] < levels[2])
            
         color_select = np.zeros_like(img[:,:,0])
         color_select[rockpix] = 1
    
         return color_select 
      ```
     
      ![Alt text](/misc/rock.png?raw=true "Title")
     
   2. ```process_image()``` analysis & worldmap creating
   
      Read in csv file as a dataframe
      ```
      import pandas as pd
      
      df = pd.read_csv('../test_dataset/robot_log.csv', delimiter=';', decimal='.')
      ```
      
      Create list of image pathnames
      ```
      csv_img_list = df["Path"].tolist()
      ```
      
      Read in ground truth map and create a 3-channel image with it
      ```
      ground_truth = mpimg.imread('../calibration_images/map_bw.png')
      ground_truth_3d = np.dstack((ground_truth*0, ground_truth*255, ground_truth*0)).astype(np.float)
      ```
      Creating a class to be the data container that read in saved data from csv file and populate this object.
      Worldmap is instantiated as 200 x 200 grids corresponding to a 200m x 200m space (same size as the ground truth
      map: 200 x 200 pixels) 
      ```
      class Databucket():
         def __init__(self):
            self.images = csv_img_list  
            self.xpos = df["X_Position"].values
            self.ypos = df["Y_Position"].values
            self.yaw = df["Yaw"].values
            self.count = 0 # This will be a running index
            self.worldmap = np.zeros((200, 200, 3)).astype(np.float)
            self.ground_truth = ground_truth_3d
        
      ```
      Instantiate a Databucket()
      ```
      data = Databucket()
      ```


Autonomous Navigation and Mapping 

   1. ```perception_step()``` and ```decision_step()``` explanation and why these functions were modified as they were.
   
   2. Autonomousmode navigation results explanation and how we might improve them.

