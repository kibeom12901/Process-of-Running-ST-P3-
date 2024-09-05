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
    bash scripts/train_perceive.sh stp3/configs/nuscenes/Perception.yml /data/Nuscene
    ```
- Training approximately took 5 days with one Nvidia GPU.
- The model was trained up to Epoch 19, accumulating data from epochs 1.

  <img src="https://github.com/user-attachments/assets/a560bea8-098f-4ed5-a1aa-a8c122024e0d" alt="Perception Training Progress" width="600">

### Prediction Module Training
- Began training the prediction module with the following command:
    ```bash
    bash scripts/train_prediction.sh stp3/configs/nuscenes/Prediction.yml data/Nuscenes tensorboard_logs/09August2024at13_52_16KST_SimulationPC_Perception/default/version_0/checkpoints/epoch=19-step=174159.ckpt
    ```

    <img src="https://github.com/user-attachments/assets/21e96725-5941-4ede-bf3d-2e5ec5835c1b" alt="Prediction Training Progress" width="600">

### Entire Model E2E Training
- Encountered a memory issue during this process, which was resolved by reducing the batch size to 1.
- Performed Planning model:
  - Once using the perception-trained model with 2 epochs to test an extreme case.
  - Once using the perception-trained model with 19 epochs.
  - Once using the prediction-trained model with 19 epochs.
- The purpose was to compare and analyze differences between the models.

### Evaluation with Entire E2E Trained Model
- Executed the following command to start Evaluation:
    ```bash
    bash scripts/eval_plan.sh tensorboard_logs/21August2024at10_25_26KST_SimulationPC2_Planning/default/version_0/checkpoints/last.ckpt data/Nuscenes
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
      
      **Sample Image of HD map**
      
      [Rest of the HD maps are here](https://drive.google.com/drive/folders/1ejEUI5i4BnId_sAtOOkq8c-44bW-Y0yN)

#### Evaluation of Perception-Trained Model with 19 Epochs 

For more details and updates, you can refer to the official [ST-P3 GitHub repository](https://github.com/OpenDriveLab/ST-P3).
