## Progress Log

### 8/5/2024 Onwards - ST-P3 Workflow

<img width="822" alt="Screenshot 2024-08-26 at 4 57 43 PM" src="https://github.com/user-attachments/assets/21e01630-ca95-402f-baba-59b0b5e9aa7a">
- **ST-P3 Framework Overview:**
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


**Data Generation for Town01_Tiny:**
  - Downloaded the entire NuScenes dataset. Using [this link](https://www.nuscenes.org/login?prevpath=download&prevhash=).


- **Training the Perception Module:**
  - Executed the following command to start Perception Module Pretraining:
    ```bash
    bash scripts/train_perceive.sh stp3/configs/nuscenes/Perception.yml /data/Nuscene
    ```
  - Training approximately took 5 days with one Nvidia GPU.
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
    - [Images For HD maps](https://drive.google.com/drive/folders/1ejEUI5i4BnId_sAtOOkq8c-44bW-Y0yN)

  
 
For more details and updates, you can refer to the official [ST-P3 GitHub repository](https://github.com/OpenDriveLab/ST-P3).

