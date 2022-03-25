# FSL-pretrained-models

In this repo, we provide a collection of scripts for pretraining models for few-shot learning, some of them are used in **[Label Hallucination for Few-Shot Classification](https://arxiv.org/abs/2112.03340)**. Thanks [SKD](https://github.com/brjathu/SKD) and [Rizve et al.](https://github.com/nayeemrizve/invariance-equivariance) for their original implementation.

## Download Data
The data we used here is preprocessed by the repo of [MetaOptNet](https://github.com/kjunelee/MetaOptNet), Please find the renamed versions of the files in below link by [RFS](https://github.com/WangYueFt/rfs).

Download and unzip the dataset, put them under ```data``` directory.

## Invariant and Equivariant Representations (IER)
We modify the `train.py` mainly to remove wandb usage and save the model every epoch to disk. Further to work with the latest pytorch version, we change the `.view()` function in `utils` to `.reshape()`.

First creating a conda environment named IER
```
conda create -n IER python=3.6
pip install -r requirements.txt
```
For the GPU we use (RTX-A100 or A6000), it requires pytorch installation of CUDA11, thus further reinstall pytorch by
```
conda install pytorch torchvision torchaudio cudatoolkit=11.3 -c pytorch
```
training the model follow the original implementation (e.g. for FC100)
```
python3 train.py --model resnet12 --model_path save --dataset FC100 --data_root data --n_aug_support_samples 5 --n_ways 5 --n_shots 1 --epochs 65 --lr_decay_epochs 60 --gamma 1.0 --contrast_temp 1.0 --mvavg_rate 0.99 --memfeature_size 64 --batch_size 64 --tags FC100,INV_EQ
```

## Self-supervised Knowledge Distillation (SKD)
We use the same environment as IER. To make the code run, we have to make a few modifications and corrections, i.e, in orginal `train_distillation.py`
```
# inputs_all = torch.cat((x, x_180, x_90, x_270),0)
inputs_all = torch.cat((x, x_90),0)

# (_,_,_,_, feat_s_all), (logit_s_all, rot_s_all)  = model_s(inputs_all[:4*batch_size], rot=True)
(_,_,_,_, feat_s_all), (logit_s_all, rot_s_all)  = model_s(inputs_all[:2*batch_size], rot=True)

# loss = loss_div + opt.gamma*loss_a / 3
loss = loss_div + opt.gamma*loss_a 
```

## Note
We found that results vary (and sometimes a lot) across different runs. To try to get models matching results for what reported in these papers, we found that it's important to have multiple runs of the initial training (generation 0 in IER and SKD) and pick the best model to start with, for distillation (generation 1). 