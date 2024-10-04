# Progress Log

## 8/5/2024 Onwards - ST-P3 Workflow

![ST-P3 Workflow](https://github.com/user-attachments/assets/21e01630-ca95-402f-baba-59b0b5e9aa7a)

### ST-P3 Framework Overview
- **Input:** Utilizes multiple camera views (`T-step N-view Camera`) capturing the environment around the vehicle over time.
- **High-Level Command:** Directs the vehicle’s actions, such as "Go Straight."

- **Backbone Network:**
  - Processes the camera inputs to extract front-view vision features. 

- **Perception Module:**
  - Aligns and aggregates features in 3D space to create **Aligned BEV Features** (Bird’s Eye View) at different time steps (`t = 1` to `t = T`), preserving geometric information.

- **Prediction Module:**
  - Generates future scene representations over multiple time horizons (`H horizons`), using past variations to predict future outcomes.
 
- **Planning Module:**
  - Creates the final vehicle trajectory (**SDV Trajectories**) by integrating BEV features, scene predictions, and front-view vision features, guided by high-level commands.
     
Further information can be found in my [ST-P3 Overview Summary](https://github.com/kibeom12901/Overview-of-ST-P3-A-Unified-Vision-Based-Autonomous-Driving-Framework).

And more detailed command Guide can be found in my [ST-P3 Command Guide](https://github.com/kibeom12901/ST-P3_Command_Guide/blob/main/README.md).

### Data Generation for E2E Training
- Downloaded the entire NuScenes dataset using [this link](https://www.nuscenes.org/login?prevpath=download&prevhash=).

### Training the Perception Module
- Executed the following command to start Perception Module Pretraining:
    ```bash
    bash scripts/train_perceive.sh ${configs} ${dataroot}
    ```
- Training approximately took 5 days with one Nvidia GPU.
- The model was trained up to Epoch 19, accumulating data from epochs 0.

  <img src="https://github.com/user-attachments/assets/a560bea8-098f-4ed5-a1aa-a8c122024e0d" alt="Perception Training Progress" width="600">

### Prediction Module Training
- Began training the prediction module with the following command:
    ```bash
    bash scripts/train_prediction.sh ${configs} ${dataroot} ${pretrained}
    ```

    <img src="https://github.com/user-attachments/assets/21e96725-5941-4ede-bf3d-2e5ec5835c1b" alt="Prediction Training Progress" width="600">

### Planning - Entire Model E2E Training
- Encountered a memory issue during this process, which was resolved by reducing the batch size to 1.
- Execute Planning Command:

    ```bash
    bash scripts/train_plan.sh ${configs} ${dataroot} ${pretrained}
    ```
    
- Performed Planning model:
  - Once using the perception-trained model with 2 epochs to test an extreme case.    
  - Once using the prediction-trained model with 20 epochs.


- The purpose was to compare and analyze differences between the models.

### Evaluation with Entire E2E Trained Model
- Executed the following command to start Evaluation:
    ```bash
    bash scripts/eval_plan.sh ${checkpoint} ${dataroot}
    ```

#### Evaluation of Perception-Trained Model with 2 Epochs (Extreme Case)

- The evaluation results show that the model’s performance varies across different metrics:
  - **Vehicle IoU:** The model shows moderate accuracy in identifying vehicles in the scene.
  - **Pedestrian IoU:** The model struggles with detecting pedestrians, as indicated by a lower IoU score.
  - **Lane Divider IoU:** The accuracy of lane divider detection is moderate, but there's room for improvement.
  - **Drivable Area IoU:** The model performs relatively well in identifying drivable areas, with a higher IoU score compared to other classes.
  - **Planning Metrics:**
    - The model demonstrates low collision probabilities with objects and bounding boxes within the first 1 to 3 seconds, indicating relatively safe trajectory planning.
    - However, the L2 distance error increases over time, suggesting that the predicted trajectories deviate more from the ground truth as the prediction horizon extends.

      <img src="https://github.com/user-attachments/assets/dfcbd05d-707c-473b-846a-e1451e43eb13" alt="Evaluation Results" width="600">
      
      **Performance Metrics of Perception Module After 2 Epochs of Training on the ST-P3 Autonomous Driving Model**

      <img width="1195" alt="Screenshot 2024-08-28 at 10 20 56 AM" src="https://github.com/user-attachments/assets/d652041a-485a-4b27-8f2f-eb71014df7b4">
      
      **Sample Images of HD map**
      
      [Rest of the HD maps are here](https://drive.google.com/drive/folders/1ejEUI5i4BnId_sAtOOkq8c-44bW-Y0yN)


#### Evaluation of Prediction-Trained Model with 20 Epochs 

- The evaluation results demonstrate the model's ability to handle various tasks related to autonomous driving, showing improvements over the perception-trained model in several key areas:

  - **Vehicle IoU:**
    The model performs moderately well in identifying vehicles in the scene, with a **Vehicle IoU of 0.3653**, which is an improvement over the perception-trained model's IoU of 0.2915. This indicates that the prediction-trained model is more accurate in detecting and understanding the position of vehicles. 

  - **Pedestrian IoU:**
    The pedestrian detection performance saw a slight decrease, with a **Pedestrian IoU of 0.1142** compared to 0.1270 from the perception-trained model. This suggests that while the model struggles more with identifying pedestrians, it still maintains reasonable performance.

  - **Lane Divider IoU:**
    The model improved significantly in identifying lane dividers, achieving a **Lane Divider IoU of 0.3221** compared to 0.2738 from the perception-trained model. This indicates that the prediction-trained model is better at recognizing lane boundaries, crucial for safe driving.

  - **Drivable Area IoU::**
    The model performs well in identifying drivable areas, with a **Drivable Area IoU of 0.7139**, which is higher than the perception-trained model’s score of 0.6210. This improvement shows that the prediction-trained model is better at understanding where the vehicle can safely drive.

    <img width="600" alt="Screenshot 2024-10-04 at 2 06 28 PM" src="https://github.com/user-attachments/assets/fb70d7aa-fba9-417e-b4c7-7271f76f4a50">
  
    **Evaluation Metrics of Prediction Module After 20 Epochs of Training on the ST-P3 Autonomous Driving Model**
  
    **[Images of HD map](https://drive.google.com/drive/folders/1ejEUI5i4BnId_sAtOOkq8c-44bW-Y0yN)**

- **Planning Metrics**:

  The planning module's ability to predict vehicle trajectories and avoid obstacles is evaluated with several metrics:

  - **Plan Object Collision (1s, 2s, 3s):**
  
    The collision probabilities with objects remain very low, particularly within the first few seconds of trajectory planning. For example, the **Plan Object Collision at 2 seconds is 0.0011**, lower than the perception-trained model’s 0.0044. This indicates that the prediction-trained model plans safer trajectories by avoiding obstacles earlier in the prediction horizon.

  - **Plan L2 Distance Error (1s, 2s, 3s):**

   The L2 distance error measures how much the predicted trajectory deviates from the ground truth over time. The model's Plan L2 Error shows improvement across all time horizons, with:
    - 1 second: **1.3312** (improved from 1.6208 in the perception-trained model)
    - 2 seconds: **2.1038** (improved from 2.4952)
    - 3 seconds: **2.8929** (improved from 3.0842)

      This consistent improvement demonstrates that the prediction-trained model provides more accurate trajectory predictions over time compared to the perception-trained model.

- **Overall Comparison:**
  - **Better Vehicle and Lane Detection:**
    The prediction-trained model improves vehicle and lane detection accuracy, essential for safely navigating complex environments.
    
  - **Safer Trajectories:**
    Lower collision probabilities and reduced L2 error indicate that the model generates safer and more accurate trajectories, particularly in avoiding obstacles and staying on the correct path.
    
  - **Slight Drop in Pedestrian Detection:**
    The model's pedestrian detection performance slightly worsens, though the impact is minimal relative to the improvements in other areas.
    
For more details and updates, you can refer to the official [ST-P3 GitHub repository](https://github.com/OpenDriveLab/ST-P3).
