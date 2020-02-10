# Adversarial-Data-Encryption

PyTorch implementation of paper "Adversarial Data Encryption"

Our implementation is based on these repositories:

- [robustness](https://github.com/MadryLab/robustness)

- [CIFAR-ZOO](https://github.com/BIGBALLON/CIFAR-ZOO)

This implementation contains the main experiments on [CIFAR-10](http://www.cs.toronto.edu/~kriz/cifar.html?usg=alkjrhjqbhw2llxlo8emqns-tbk0at96jq) dataset.

### Abstract

In the big data era, many organizations face the dilemma of data sharing. Regular data sharing is often necessary  for human-centered discussion and communication, especially in medical scenarios. 
However, unprotected data sharing may also lead to  data leakage. Inspired by adversarial attack, 
we propose a method for data encryption. Encrypted images look identical to original ones for human beings, which means they can be used for discussion. However, for mechine learning algorithms, especially encoders, encrypted images are misleading, based on which malicious data stealers cannot obtain effective models for actual use. (Thus, our encryption algorithm is essential for safely data sharing.)

<img src="https://github.com/Alxead/Adversarial-Data-Encryption/blob/master/images/mainfig.png" width="600" alt="mainfig"/>

## Getting Started

### Requirements

- Python (**>=3.6**)
- PyTorch (**>=1.1.0**)
- Tensorboard(**>=1.4.0**) (for ***visualization***)
- Other dependencies (robustness, pyyaml, easydict)

```
pip install -r requirements.txt
```

`robustness` is a package [MadryLab](http://madry-lab.ml/) created to make training, evaluating, and exploring neural networks flexible and easy.  We mainly use `robustness` in the next first step (1. train a base classifier) and second step (2. encrypt data) . 

### 1. Train a base classifier

First download CIFAR-10 and put it in an appropriate directory (e.g.  ``./data/cifar10``). Then train a standard (not robust) ResNet-50 as base classifier through the following command:

```
python -m robustness.main --dataset cifar --data ./data/cifar10 --adv-train 0 \
--arch resnet50 --out-dir ./logs/checkpoints/dir/ --exp-name resnet50
```

After training, the base classifier is saved at  ``./logs/checkpoints/dir/resnet50/checkpoint.pt.best`` ,it will be used to encrypt the data.

### 2. Encrypt data

To encrypt the original CIFAR-10, simply run:

```
python encrypt.py --orig-data ./data/cifar10 --enc-data ./data \
--resume-path ./logs/checkpoints/dir/resnet50/checkpoint.pt.best --enc-method basic
```

Use `--orig-data` to specify the directory where original CIFAR-10 is saved. Use `--enc-data` to specify the directory where encrypted CIFAR-10 will be saved.  Resume the base classifier from `--resume-path` and use option `--enc-method` to specify the encryption method. We provide four encryption methods: `basic`, `mixup`, `horiz`, `mixandcat`. Encrypted data will be named with a suffix of encryption method. The other parameters of the encryption process are set to the values used in our paper by default. If you want to change them, you can check `encrypt.py` for more details.

### 3. Validate Encryption method

To verify if the encryption method is useful, you should train a model using the encrypted data, and then observe its performance on the original test set and the encrypted test set. You can do this through the following command:

```
python train.py --work-path ./experiments/cifar10/preresnet110
```

This code trains a PreResNet-110 using the encrypted data. Note that before training, first fill in the path of the encrypted data and original data in `config.yaml`. We use yaml file `config.yaml` to save all the parameters during training, check files in `./experimets` for more details.

At the beginning of training, you will see that the accuracy of the original test set is similar to that of the encrypted test set, but as the training progresses, the accuracy on the original test set will become extremely low.

## Experimental Results

Classification accuracy of different models on the CIFAR-10 encrypted test set and original test set.

- Basic encryption method

| model          | original test set acc | encrypted test set acc |
| -------------- | --------------------- | ---------------------- |
| DenseNet-100bc | 22.78%                | 94.70%                 |
| PreResNet-110  | 20.67%                | 94.64%                 |
| VGG-19         | 28.77%                | 93.58%                 |

- Horizontal Concat method

| model          | original test set acc | encrypted test set acc |
| -------------- | --------------------- | ---------------------- |
| DenseNet-100bc | 29.69%                | 94.62%                 |
| PreResNet-110  | 32.65%                | 94.49%                 |
| VGG-19         | 48.13%                | 94.30%                 |

- Mixup And Concat method

| model          | original test set acc | encrypted test set acc |
| -------------- | --------------------- | ---------------------- |
| DenseNet-100bc | 32.92%                | 94.45%                 |
| PreResNet-110  | 37.21%                | 94.03%                 |
| VGG-19         | 55.00%                | 93.06%                 |

## Citation

* TODO: cite them -> [MadryLab](http://madry-lab.ml/)

