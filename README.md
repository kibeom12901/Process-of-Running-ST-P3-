## Progress Log

<img width="822" alt="Screenshot 2024-08-26 at 4 57 43 PM" src="https://github.com/user-attachments/assets/21e01630-ca95-402f-baba-59b0b5e9aa7a">

### 8/5/2024 Onwards - ST-P3 Workflow

**Data Generation for Town01_Tiny:**
  - Initially attempted to generate data for `Town01_Tiny`.
  - Created a new directory to fix an error related to the data path.
  - Upon further review, identified that the error was due to an incorrect directory path within the data folder, where a separate data path was needed for NuScenes.
  - Downloaded the entire NuScenes dataset to correct the issue. Using [this link](https://www.nuscenes.org/login?prevpath=download&prevhash=).


- **Training the Perception Module:**
  - Executed the following command to start Perception Module Pretraining:
    ```bash
    bash scripts/train_perceive.sh stp3/configs/nuscenes/Perception.yml /data/Nuscene
    ```
  - Training approximately took 5 days with one Nvidia GPU.
  - The entire training process is expected to take 8-9 days, covering 41 epochs.
  - The model gets trained up to Epoch 19, accumulating data from epochs 1
    
    <img width="600" alt="Screenshot 2024-08-22 at 11 20 10 AM" src="https://github.com/user-attachments/assets/a560bea8-098f-4ed5-a1aa-a8c122024e0d">
    
- **Prediction Module Training:**
  - Began training the prediction module with the following command:
    ```bash
    bash scripts/train_prediction.sh stp3/configs/nuscenes/Prediction.yml data/Nuscenes tensorboard_logs/09August2024at13_52_16KST_SimulationPC_Perception/default/version_0/checkpoints/epoch=19-step=174159.ckpt
    ```
    <img width="600" alt="Screenshot 2024-08-22 at 11 21 21 AM" src="https://github.com/user-attachments/assets/21e96725-5941-4ede-bf3d-2e5ec5835c1b">

- **Entire Model E2E training:**
  - Encountered a memory issue during this process, which was resolved by reducing the batch size to 1.
  - Performed Planning model:
    - Once using the perception trained model with 2 epochs to test extreme case.
    - Once using the perception-trained model with 19 epochs.
    - Once using the prediction-trained model with 19 epochs.
  - The purpose was to compare and analyze differences between the models.
 
- **Evluation with Entire E2E trained Model :**
  - Executed the following command to start Evluation:
    ```bash
    bash scripts/eval_plan.sh tensorboard_logs/21August2024at10_25_26KST_SimulationPC2_Planning/default/version_0/checkpoints/last.ckpt data/Nuscenes
    ```
  - The result contains the following metrics:

    - **IoU Metrics:**
      - **`vehicle_iou`:** This measures how well the model is identifying vehicles in the scene by comparing the predicted vehicle segments with the actual vehicles present.
      - **`pedestrian_iou`:** This evaluates how accurately the model is detecting and segmenting pedestrians within the environment.
      - **`lane_divider_iou`:** This assesses the model's ability to correctly identify lane dividers on the road by comparing predicted lane segments with the ground truth.
      - **`drivable_area_iou`:** This measures how well the model is predicting drivable areas, evaluating the overlap between the predicted drivable regions and the actual drivable space.

    - **Planning Metrics:**
      - **`plan_obj_col_1s`:** Represents the probability of the vehicle's planned trajectory colliding with an object (such as other vehicles or obstacles) within the first second of planning. Lower values indicate safer trajectories.
      - **`plan_obj_box_col_1s`:** Similar to `plan_obj_col_1s`, this metric specifically measures the probability of collision with the bounding boxes of objects within the first second.
      - **`plan_L2_1s`:** Indicates the L2 distance error between the planned trajectory and the ground truth trajectory after 1 second. It measures how much the planned path deviates from the ideal path.
      - **`plan_obj_col_2s`:** Represents the probability of the planned trajectory colliding with an object within 2 seconds. This metric shows how the risk of collision evolves as the planning horizon extends.
      - **`plan_obj_box_col_2s`:** Similar to `plan_obj_col_2s`, but focused on the probability of collision with bounding boxes of objects after 2 seconds.
      - **`plan_L2_2s`:** Measures the L2 distance error between the planned and actual trajectory after 2 seconds. This shows how accurately the model predicts the vehicle's position further into the future.
      - **`plan_obj_col_3s`:** Represents the probability of the planned trajectory colliding with an object within 3 seconds. This metric assesses the risk of collision over a longer planning horizon.
      - **`plan_obj_box_col_3s`:** Similar to `plan_obj_col_3s`, this measures the probability of collision with bounding boxes after 3 seconds.
      - **`plan_L2_3s`:** Indicates the L2 distance error between the planned trajectory and the ground truth after 3 seconds, showing how the accuracy of the trajectory prediction changes over time.
  
    - **Image Explanation:**
      - **Camera Views:** The images include multiple camera views (e.g., `FRONT`, `FRONT LEFT`, `FRONT RIGHT`, `BACK`, etc.) from the autonomous vehicle. These views display the actual scene as captured by the vehicle's cameras during the evaluation process.
        - These camera views help visualize what the model "sees" and how it interprets the environment in real-time.
      
      - **Top-Down View with HD Map Overlay:** The top-down view on the right side of the image shows an HD map with overlaid predictions.
        - **Lane Dividers:** Represented typically in a specific color (e.g., blue), showing where the model predicts lane dividers to be.
        - **Drivable Area:** Another color (e.g., gray) shows where the model predicts drivable areas in the environment.
        - **Vehicle and Pedestrian Segmentation:** Different colors (e.g., orange for vehicles and blue for pedestrians) indicate where the model has detected vehicles and pedestrians in the scene.
        - **Trajectory Visualization:** The predicted vehicle trajectory is often shown as a path overlaid on this map, comparing it to the ground truth trajectory to evaluate accuracy.
      
      - **Purpose of Visuals:** 
        - These visuals allow you to assess how well the model’s predictions align with the actual scene, providing a qualitative understanding of the model’s performance in real-world scenarios.
        - You can compare these visual predictions against the IoU and planning metrics to get a comprehensive view of the model's effectiveness.

  - **Evaluation of perception trained model with 2 epochs (extreme case):**
    - The evaluation results show that the model’s performance varies across different metrics:
      - **Vehicle IoU:** The model shows moderate accuracy in identifying vehicles in the scene.
      - **Pedestrian IoU:** The model struggles with detecting pedestrians, as indicated by a lower IoU score.
      - **Lane Divider IoU:** The accuracy of lane divider detection is moderate, but there's room for improvement.
      - **Drivable Area IoU:** The model performs relatively well in identifying drivable areas, with a higher IoU score compared to other classes.
      - **Planning Metrics:** 
        - The model demonstrates low collision probabilities with objects and bounding boxes within the first 1 to 3 seconds, indicating relatively safe trajectory planning.
        - However, the L2 distance error increases over time, suggesting that the predicted trajectories deviate more from the ground truth as the prediction horizon extends.
        <img width="350" alt="Screenshot 2024-08-22 at 1 39 07 PM" src="https://github.com/user-attachments/assets/dfcbd05d-707c-473b-846a-e1451e43eb13">
    - [Images](https://drive.google.com/drive/folders/1ejEUI5i4BnId_sAtOOkq8c-44bW-Y0yN)

  
 
For more details and updates, you can refer to the official [ST-P3 GitHub repository](https://github.com/OpenDriveLab/ST-P3).

