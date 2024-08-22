<img width="1440" alt="Screenshot 2024-08-22 at 11 19 09 AM" src="https://github.com/user-attachments/assets/b662eedf-701c-4a7d-a9d4-b0142d509337"># Process-of-Running-ST-P3

## Progress Log

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
  - The model gets trained up to Epoch 19, accumulating data from epochs 1-
  - 
<img width="600" alt="Screenshot 2024-08-22 at 11 20 10 AM" src="https://github.com/user-attachments/assets/a560bea8-098f-4ed5-a1aa-a8c122024e0d">


- **Prediction Module Training:**
  - Began training the prediction module with the following command:
    ```bash
    bash scripts/train_prediction.sh stp3/configs/nuscenes/Prediction.yml data/Nuscenes tensorboard_logs/09August2024at13_52_16KST_SimulationPC_Perception/default/version_0/checkpoints/epoch=19-step=174159.ckpt
    ```
<img width="600" alt="Screenshot 2024-08-22 at 11 21 21 AM" src="https://github.com/user-attachments/assets/21e96725-5941-4ede-bf3d-2e5ec5835c1b">

- **Entiner Model E2E training:**
  - Encountered a memory issue during this process, which was resolved by reducing the batch size to 1.
  - Performed evaluation twice:
    - Once using the pretrained model.
    - Once using the prediction-trained model.
  - The purpose was to compare and analyze differences between the models.
  
 
For more details and updates, you can refer to the official [ST-P3 GitHub repository](https://github.com/OpenDriveLab/ST-P3).

