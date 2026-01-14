# DiffusionDrive Training and Evaluation

## 1. Cache dataset for faster training and evaluation
```bash
# cache dataset for training
python navsim/planning/script/run_dataset_caching.py agent=diffusiondrivev2_rl_agent experiment_name=diffusiondrivev2_cache train_test_split=navtrain

# cache dataset for evaluation
python navsim/planning/script/run_metric_caching.py train_test_split=navtest cache.cache_path=$NAVSIM_EXP_ROOT/metric_cache

# cache dataset for calculating PDMS during training.
python navsim/planning/script/run_metric_caching.py train_test_split=navtrain cache.cache_path=$NAVSIM_EXP_ROOT/train_pdm_cache

# cache dataset for fast evaluation (optional)
python navsim/planning/script/run_dataset_caching.py agent=diffusiondrivev2_rl_agent experiment_name=diffusiondrivev2_cache train_test_split=navtest cache_path=$NAVSIM_EXP_ROOT/metric_feature_cache

```
## 2. Download Checkpoints
Download the following checkpoints from [DiffusionDriveV2](https://huggingface.co/hustvl/DiffusionDriveV2), [DiffusionDrive](https://huggingface.co/hustvl/DiffusionDrive) and [resnet34.a1_in1k](https://huggingface.co/timm/resnet34.a1_in1k),  and place them in the `ckpts` directory:
- `resnet34.a1_in1k`
- `diffusiondrive_navsim_88p1_PDMS` (DiffusionDrive model)
- `diffusiondrivev2_rl.ckpt` (The RL-trained DiffusionDrive model.)
- `diffusiondrivev2_sel.ckpt` (diffusiondrivev2_rl + mode selector)


## 3. Training
```bash
# Training DiffusionDrive with reinforcement learning
python $NAVSIM_DEVKIT_ROOT/navsim/planning/script/run_training.py \
        agent=diffusiondrivev2_rl_agent \
        experiment_name=training_diffusiondrivev2_rl_agent \
        train_test_split=navtrain \
        agent.checkpoint_path=ckpts/diffusiondrive_navsim_88p1_PDMS \
        split=trainval \
        trainer.params.max_epochs=10 \
        cache_path="${NAVSIM_EXP_ROOT}/training_cache/" \
        use_cache_without_dataset=True \
        force_cache_computation=False \ 
        +df_verssion=v2

# Training mode selector
python $NAVSIM_DEVKIT_ROOT/navsim/planning/script/run_training.py \
        agent=diffusiondrivev2_sel_agent \
        experiment_name=training_diffusiondrivev2_sel_agent \
        train_test_split=navtrain \
        agent.checkpoint_path=ckpts/diffusiondrivev2_rl.ckpt \
        split=trainval \
        trainer.params.max_epochs=20 \
        cache_path="${NAVSIM_EXP_ROOT}/training_cache/" \
        use_cache_without_dataset=True \
        force_cache_computation=False \
        +df_verssion=v2

```

if you want train DiffusionDrive v1, use this command
```
python $NAVSIM_DEVKIT_ROOT/navsim/planning/script/run_training.py \
        agent=diffusiondrive_agent \
        experiment_name=training_diffusiondrive_agent  \
        train_test_split=mini \
        split=mini\
        trainer.params.max_epochs=100 \
        cache_path="${NAVSIM_EXP_ROOT}/training_cache/" \
        use_cache_without_dataset=True  \
        force_cache_computation=False  \
        +df_version=v1

```

## 4. Evaluation
Use the following command to evaluate the trained model rapidly (**several times faster than the official evaluation script**):
```bash
python $NAVSIM_DEVKIT_ROOT/navsim/planning/script/run_pdm_score_fast.py \
        agent=diffusiondrivev2_sel_agent \
        experiment_name=diffusiondrivev2_agent_eval \
        train_test_split=navtest \
        agent.checkpoint_path=ckpts/diffusiondrivev2_sel.ckpt \
        +metric_cache_path="${NAVSIM_EXP_ROOT}/metric_cache/" \
        +test_cache_path="${NAVSIM_EXP_ROOT}/metric_feature_cache/"
```

Alternatively, you can use the official evaluation script, but you will need to uncomment this [line](https://github.com/hustvl/DiffusionDriveV2/blob/master/navsim/agents/diffusiondrivev2/diffusiondrivev2_model_sel.py#L1491).
```bash
python $NAVSIM_DEVKIT_ROOT/navsim/planning/script/run_pdm_score.py \
        train_test_split=navtest \
        agent=diffusiondrivev2_sel_agent \
        worker=ray_distributed \
        agent.checkpoint_path=ckpts/diffusiondrivev2_sel.ckpt \
        experiment_name=diffusiondrivev2_agent_eval
```

