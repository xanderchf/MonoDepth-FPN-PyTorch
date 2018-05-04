# I2D: Single Image Depth Estimation with FPN

[![License][license]][license-url]

A simple end-to-end model with fast convergence achieves state-of-the-art performance in depth prediction. This is a class project for Advanced Computer Vision (CS 7476) at Georgia Tech. In this project, we used a Feature Pyramid Network (FPN) backbone to estimate depth map by taking a single RGB image. We tested the performance of our model on the NYU Depth V2 Dataset and the KITTI Dataset.

## Requirements

* Python 3
* Jupyter Notebook (for visualization)
* PyTorch 
  * Tested with PyTorch 0.3.0.post4
* CUDA 8 (if using CUDA)

## Data Processing
### NYU Depth V2 Dataset
* The NYU Depth V2 dataset contains a variety of indoor scenes, with 249 scenes for training and 215 scenes for testing. We used the official split for training and testing.
* Following previous works, we used the official toolbox which uses the Colorization method proposed by Levin et al. to fill in the missing values of depth map in the training set.
* Following previous works, we evaluated our model using the official evaluation set of 654 densely labeled image pairs.


### KITTI Dataset
* The KITTI dataset consists of 61 outdoor scenes with “city”, “road”, and “residential” categories.
* Following previous works, we used the same to fill in the missing values of the sparse depth maps in the training set.
* To compare with performances of previous studies, we employed the evaluation split used by Eigen et al.

## Model

<p align="center"><img src='https://github.com/xanderchf/i2d/blob/master/architecture.png' width=600></p>

* Used Feature Pyramid Network (FPN) with ResNet101 as backbone (shaded yellow), loaded ImageNet pretrained weight.
* Used pixel shuffle for upsampling and fuse feature maps with add operation; bilinear interpolation is employed after pixel shuffle to tackle the problem of inconsistent size.
* Two consecutive 3x3 convolutions for feature processing.
* No linearity in the top-down branch of FPN, and ReLU in other convolution layers, and Sigmoid in the prediction layer for better stability.
* Trained on the weighted sum of the depth loss, the gradient loss, and the normal loss for 20 
epochs; gradient loss added after epoch 1, and normal loss added after epoch 10.
* Outputs prediction of size ¼, and evaluated after bilinear upsampling.

## Loss Function

We employed three parts in the loss function in our model. The loss is a weighted sum of 3 parts: the depth loss, the gradient loss and the surface normal loss.

### Depth Loss
![img](https://latex.codecogs.com/gif.latex?L_%7B%5Ctextup%7Bdepth%7D%7D%20%3D%20%5Cfrac%7B1%7D%7Bn%7D%20%5Csum_%7Bi%3D1%7D%5E%7Bn%7D%20%5Csqrt%5B%5D%7B%5Clog%5E2%28d_i%29%20-%20%5Clog%5E2%28p_i%29%7D)

The depth loss is RMSE in log scale, which we found converges better than L1 and L2 loss.Supervising in log scale makes the classifier focus more on near objects.

### Gradient Loss
![img](https://latex.codecogs.com/gif.latex?L_%7B%5Ctextup%7Bgrad%7D%7D%20%3D%20%5Cfrac%7B1%7D%7Bn%7D%20%5Csum_%7Bi%3D1%7D%5E%7Bn%7D%20%5Cbig%7C%5Cbig%7C%20%5Cnabla%20%5Ctextbf%7Bd%7D%20-%20%5Cnabla%20%5Ctextbf%7Bp%7D%20%5Cbig%7C%5Cbig%7C_1)

The gradient of depth maps is obtained by a Sobel filter; the gradient loss is the L1 norm of the difference.

### Surface Normal Loss
![img](https://latex.codecogs.com/gif.latex?L_%7B%5Ctextup%7Bnormal%7D%7D%20%3D%20%5Cfrac%7B1%7D%7Bn%7D%20%5Csum_%7Bi%3D1%7D%5E%7Bn%7D%20%5Cbigg%28%201%20-%20%5Cfrac%7B%5Clangle%20n%5Ed_i%2C%20n%5Ep_i%20%5Crangle%7D%7B%7C%7Cn%5Ed_i%7C%7C%5Ctextbf%7B%20%7D%7C%7Cn%5Ep_i%7C%7C%7D%20%5Cbigg%29)

We also employed the normal vector loss proposed by Hu et al., which helps refining details.

## To Run

```
python3 main_fpn.py --cuda --bs 6
```
To continue training from a saved model, use
```
python3 main_fpn.py --cuda --bs 6 --r True --checkepoch 10
```
To visualize the reconstructed data, run the jupyter notebook in vis.ipynb.

## To-dos

- [x] Add visualization ipynb
- [ ] Add support for the ScanNet Dataset
- [ ] Add code for CVPR 2018 ROB Challenge

<!-- Markdown link & img dfn's -->
[license]: https://img.shields.io/github/license/mashape/apistatus.svg
[license-url]: https://github.com/xanderchf/i2d/blob/master/LICENSE

## References
* Silberman, Nathan, et al. "Indoor segmentation and support inference from rgbd images." European Conference on Computer Vision. Springer, Berlin, Heidelberg, 2012.
* Geiger, Andreas, et al. "Vision meets robotics: The KITTI dataset." The International Journal of Robotics Research 32.11 (2013): 1231-1237.
* Eigen, D., Puhrsch, C., Fergus, R.: Depth map prediction from a single image using a multiscale
deep network. In: Advances in neural information processing systems (NIPS). (2014)
2366–2374
* Levin, Anat, Dani Lischinski, and Yair Weiss. "Colorization using optimization." ACM Transactions on Graphics (ToG). Vol. 23. No. 3. ACM, 2004.
* Hu, Junjie, et al. "Revisiting Single Image Depth Estimation: Toward Higher Resolution Maps with Accurate Object Boundaries." arXiv preprint arXiv:1803.08673 (2018).
