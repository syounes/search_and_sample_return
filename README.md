# search_and_sample_return
Robotics Nanodegree Program Project Submission

I'll go through the basic implementation and the necessary steps for the project submission.

First I downloaded the simulator and recorded some data then I got throught the environment setup with the python starterkit and finally I run the Rover_Project_Test_Notebook.ipynb from the project repository folder under the RoboND environmnet and Jupyter Notebook.

Notebook Analysis

   1. Obstacle and rock sample identification.
      In the perspetive transform image we mapped the navigable terrain, the obstacles that appear in dark and we identified the field of vew of camera by creating a mask
      
      ```
      def perspect_transform(img, src, dst):
           
         M = cv2.getPerspectiveTransform(src, dst)
         warped = cv2.warpPerspective(img, M, (img.shape[1], img.shape[0]))# keep same size as input image
         mask = cv2.warpPerspective(np.ones_like(img[:,:,0]), M, (img.shape[1], img.shape[0]))
         return warped, mask
     ```
   
   
   2. ```process_image()``` analysis & worldmap creating

Autonomous Navigation and Mapping 

   1. ```perception_step()``` and ```decision_step()``` explanation and why these functions were modified as they were.
   
   2. Autonomousmode navigation results explanation and how we might improve them.

