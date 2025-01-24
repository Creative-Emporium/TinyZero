See [OLD_README.md](OLD_README.md)

## Instalation
```
conda create -n zero python=3.9
pip install -e .
# install torch [or you can skip this step and let vllm to install the correct version for you]
pip install torch==2.4.0 --index-url https://download.pytorch.org/whl/cu121

# install vllm
pip3 install vllm==0.6.3 # or you can install 0.5.4, 0.4.2 and 0.3.1
pip3 install ray

# flash attention 2
pip3 install flash-attn --no-build-isolation

# quality of life
pip install wandb IPython matplotlib
```

## The Task

## Generate Data
```
conda activate zero
python examples/data_preprocess/countdown.py
```

## Run Training
```
conda activate zero
```
**Single GPU dry run**
Works for model <= 1.5B

For Qwen2.5-0.5B base, we know it fails to learn reasoning.

```
export CUDA_VISIBLE_DEVICES=7
export N_GPUS=1
export BASE_MODEL=Qwen/Qwen2.5-1.5B
export DATA_DIR=$HOME/data/countdown
export WANDB_API_KEY=0929e692448f1bc929d71d7e3ece80073c3041e6
export EXPERIMENT_NAME=countdown-qwen2.5-1.5b
export VLLM_ATTENTION_BACKEND=XFORMERS

PYTHONUNBUFFERE=1 python3 -m verl.trainer.main_ppo \
 data.train_files=$DATA_DIR/train.parquet \
 data.val_files=$DATA_DIR/test.parquet \
 data.train_batch_size=256 \
 data.val_batch_size=1312 \
 data.max_prompt_length=256 \
 data.max_response_length=1024 \
 actor_rollout_ref.model.path=$BASE_MODEL \
 actor_rollout_ref.actor.optim.lr=1e-6 \
 actor_rollout_ref.actor.ppo_mini_batch_size=128 \
 actor_rollout_ref.actor.ppo_micro_batch_size=8 \
 actor_rollout_ref.rollout.log_prob_micro_batch_size=8 \
 actor_rollout_ref.rollout.tensor_model_parallel_size=$N_GPUS \
 actor_rollout_ref.rollout.gpu_memory_utilization=0.4 \
 actor_rollout_ref.ref.log_prob_micro_batch_size=4 \
 critic.optim.lr=1e-5 \
 critic.model.path=$BASE_MODEL \
 critic.ppo_micro_batch_size=8 \
 algorithm.kl_ctrl.kl_coef=0.01 \
 trainer.logger=['wandb'] \
 +trainer.val_before_train=False \
 trainer.default_hdfs_dir=null \
 trainer.n_gpus_per_node=$N_GPUS \
 trainer.nnodes=1 \
 trainer.save_freq=100 \
 trainer.test_freq=100 \
 trainer.project_name=TinyZero \
 trainer.experiment_name=$EXPERIMENT_NAME \
 trainer.total_epochs=15 2>&1 | tee verl_demo.log
```

**3B model dry run**
In this case, the base model is able to develop sophisticated reasoning skills.
```
export CUDA_VISIBLE_DEVICES=4,5
export N_GPUS=2
export BASE_MODEL=Qwen/Qwen2.5-3B
export DATA_DIR=$HOME/data/countdown
export ROLLOUT_TP_SIZE=2
export WANDB_API_KEY=0929e692448f1bc929d71d7e3ece80073c3041e6
export EXPERIMENT_NAME=countdown-qwen2.5-3b
export VLLM_ATTENTION_BACKEND=XFORMERS

python3 -m verl.trainer.main_ppo \
 data.train_files=$DATA_DIR/train.parquet \
 data.val_files=$DATA_DIR/test.parquet \
 data.train_batch_size=256 \
 data.val_batch_size=1312 \
 data.max_prompt_length=256 \
 data.max_response_length=1024 \
 actor_rollout_ref.model.path=$BASE_MODEL \
 actor_rollout_ref.actor.optim.lr=1e-6 \
 actor_rollout_ref.actor.ppo_mini_batch_size=128 \
 actor_rollout_ref.actor.ppo_micro_batch_size=8 \
 actor_rollout_ref.rollout.log_prob_micro_batch_size=8 \
 actor_rollout_ref.rollout.tensor_model_parallel_size=$ROLLOUT_TP_SIZE \
 actor_rollout_ref.rollout.gpu_memory_utilization=0.4 \
 actor_rollout_ref.ref.log_prob_micro_batch_size=4 \
 critic.optim.lr=1e-5 \
 critic.model.path=$BASE_MODEL \
 critic.ppo_micro_batch_size=8 \
 algorithm.kl_ctrl.kl_coef=0.001 \
 trainer.logger=['wandb'] \
 +trainer.val_before_train=False \
 trainer.default_hdfs_dir=null \
 trainer.n_gpus_per_node=$N_GPUS \
 trainer.nnodes=1 \
 trainer.save_freq=100 \
 trainer.test_freq=100 \
 trainer.project_name=TinyZero \
 trainer.experiment_name=$EXPERIMENT_NAME \
 trainer.total_epochs=15 2>&1 | tee verl_demo.log
```

**OpenLlama 7B model dry run**
Will the model work?
```

export CUDA_VISIBLE_DEVICES=4,5,6,7
export N_GPUS=4
export HF_HUB_CACHE=/mnt/fs/code/research/jiayi/TinyZero/cache
export EXPERIMENT_NAME=countdown-qwen2.5-7b
export BASE_MODEL=Qwen/Qwen2.5-7B
export DATA_DIR=$HOME/data/countdown
export ROLLOUT_TP_SIZE=2
export WANDB_API_KEY=0929e692448f1bc929d71d7e3ece80073c3041e6
export VLLM_ATTENTION_BACKEND=XFORMERS

python3 -m verl.trainer.main_ppo \
 data.train_files=$DATA_DIR/train.parquet \
 data.val_files=$DATA_DIR/test.parquet \
 data.train_batch_size=256 \
 data.val_batch_size=1312 \
 data.max_prompt_length=256 \
 data.max_response_length=1024 \
 actor_rollout_ref.model.path=$BASE_MODEL \
 actor_rollout_ref.actor.optim.lr=1e-6 \
 actor_rollout_ref.actor.ppo_mini_batch_size=128 \
 actor_rollout_ref.actor.ppo_micro_batch_size=8 \
 actor_rollout_ref.rollout.log_prob_micro_batch_size=8 \
 actor_rollout_ref.rollout.tensor_model_parallel_size=$ROLLOUT_TP_SIZE \
 actor_rollout_ref.rollout.gpu_memory_utilization=0.4 \
 actor_rollout_ref.ref.log_prob_micro_batch_size=4 \
 critic.optim.lr=1e-5 \
 critic.model.path=$BASE_MODEL \
 critic.ppo_micro_batch_size=8 \
 algorithm.kl_ctrl.kl_coef=0.001 \
 trainer.logger=['wandb'] \
 +trainer.val_before_train=False \
 trainer.default_hdfs_dir=null \
 trainer.n_gpus_per_node=$N_GPUS \
 trainer.nnodes=1 \
 trainer.save_freq=100 \
 trainer.test_freq=100 \
 trainer.project_name=TinyZero \
 trainer.experiment_name=$EXPERIMENT_NAME \
 trainer.total_epochs=15 2>&1 | tee verl_demo.log
```

### Instruct Abaltion
We experiment with QWen-2.5-3B Instruct too. To follow chat template, we need to reprocess the data 

```
conda activate zero
python examples/data_preprocess/countdown.py --template_type=qwen-instruct --local_dir=$HOME/data/countdown-qwen-instruct
```
Then use this data to train the instruct model.

```
export CUDA_VISIBLE_DEVICES=4,5
export N_GPUS=2
export BASE_MODEL=Qwen/Qwen2.5-3B-Instruct
export DATA_DIR=$HOME/data/countdown-qwen-instruct
export ROLLOUT_TP_SIZE=2
export WANDB_API_KEY=0929e692448f1bc929d71d7e3ece80073c3041e6
export EXPERIMENT_NAME=countdown-qwen2.5-3b-instruct
export VLLM_ATTENTION_BACKEND=XFORMERS

python3 -m verl.trainer.main_ppo \
 data.train_files=$DATA_DIR/train.parquet \
 data.val_files=$DATA_DIR/test.parquet \
 data.train_batch_size=256 \
 data.val_batch_size=1312 \
 data.max_prompt_length=256 \
 data.max_response_length=1024 \
 actor_rollout_ref.model.path=$BASE_MODEL \
 actor_rollout_ref.actor.optim.lr=1e-6 \
 actor_rollout_ref.actor.ppo_mini_batch_size=128 \
 actor_rollout_ref.actor.ppo_micro_batch_size=8 \
 actor_rollout_ref.rollout.log_prob_micro_batch_size=8 \
 actor_rollout_ref.rollout.tensor_model_parallel_size=$ROLLOUT_TP_SIZE \
 actor_rollout_ref.rollout.gpu_memory_utilization=0.4 \
 actor_rollout_ref.ref.log_prob_micro_batch_size=4 \
 critic.optim.lr=1e-5 \
 critic.model.path=$BASE_MODEL \
 critic.ppo_micro_batch_size=8 \
 algorithm.kl_ctrl.kl_coef=0.001 \
 trainer.logger=['wandb'] \
 +trainer.val_before_train=False \
 trainer.default_hdfs_dir=null \
 trainer.n_gpus_per_node=$N_GPUS \
 trainer.nnodes=1 \
 trainer.save_freq=100 \
 trainer.test_freq=100 \
 trainer.project_name=TinyZero \
 trainer.experiment_name=$EXPERIMENT_NAME \
 trainer.total_epochs=15 2>&1 | tee verl_demo.log
```


### Reward Shaping Experiments

```
## IMPORTANT ##
export COUNTDOWN_REWARD_FUNCTION=level2
## IMPORTANT - change this to level3 and level4 ##
export HF_HUB_CACHE=/mnt/fs/code/research/jiayi/TinyZero/cache
export CUDA_VISIBLE_DEVICES=6,7
export N_GPUS=2
export BASE_MODEL=Qwen/Qwen2.5-3B
export DATA_DIR=$HOME/data/countdown
export ROLLOUT_TP_SIZE=2
export WANDB_API_KEY=0929e692448f1bc929d71d7e3ece80073c3041e6
export EXPERIMENT_NAME=countdown-qwen2.5-3b-reward-${COUNTDOWN_REWARD_FUNCTION}
export VLLM_ATTENTION_BACKEND=XFORMERS

python3 -m verl.trainer.main_ppo \
 data.train_files=$DATA_DIR/train.parquet \
 data.val_files=$DATA_DIR/test.parquet \
 data.train_batch_size=256 \
 data.val_batch_size=1312 \
 data.max_prompt_length=256 \
 data.max_response_length=1024 \
 actor_rollout_ref.model.path=$BASE_MODEL \
 actor_rollout_ref.actor.optim.lr=1e-6 \
 actor_rollout_ref.actor.ppo_mini_batch_size=128 \
 actor_rollout_ref.actor.ppo_micro_batch_size=8 \
 actor_rollout_ref.rollout.log_prob_micro_batch_size=8 \
 actor_rollout_ref.rollout.tensor_model_parallel_size=$ROLLOUT_TP_SIZE \
 actor_rollout_ref.rollout.gpu_memory_utilization=0.4 \
 actor_rollout_ref.ref.log_prob_micro_batch_size=4 \
 critic.optim.lr=1e-5 \
 critic.model.path=$BASE_MODEL \
 critic.ppo_micro_batch_size=8 \
 algorithm.kl_ctrl.kl_coef=0.001 \
 trainer.logger=['wandb'] \
 +trainer.val_before_train=False \
 trainer.default_hdfs_dir=null \
 trainer.n_gpus_per_node=$N_GPUS \
 trainer.nnodes=1 \
 trainer.save_freq=100 \
 trainer.test_freq=100 \
 trainer.project_name=TinyZero \
 trainer.experiment_name=$EXPERIMENT_NAME \
 trainer.total_epochs=15 2>&1 | tee verl_reward_${COUNTDOWN_REWARD_FUNCTION}.log
```
