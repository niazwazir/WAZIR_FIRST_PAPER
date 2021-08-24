# [3D FSRCNN for MRI Images](https://doi.org/10.1051/itmconf/20203203044)

## What is this?
**Abstract** : Single image super-resolution using deep learning techniques has shown very high reconstruction performance over the last few years. We propose a novel three-dimensional convolutional neural network called 3D FSRCNN based on FSRCNN, which reinstates the high-resolution quality of structural MRI. The 3D neural network generates output brain images of high-resolution (HR) from a low-resolution (LR) input image. A simple design ensures less time complexity and high reconstruction quality. The network is trained using T1-weighted structural MRI images from the human connectome project dataset which is a large publicly available brain MRI database.


## About files
- [2DSRCNN.ipynb](2DSRCNN.ipynb) : Implementation of 2D SRCNN for testing
- [3DSRCNN_32x32.ipynb](3DSRCNN_32x32.ipynb) : Implementation of 3D SRCNN for testing
- [3DFSRCNN_56_16.ipynb](3DFSRCNN_56_16.ipynb) : Implementation of our 3D FSRCNN
- [choosing_best_setting.ipynb](choosing_best_setting.ipynb) : Implemented different settings of our model to find the best.
- [testing.ipynb](testing.ipynb) : Notebook which has testing results for all three models.
- [history_56_16_relu_2.p](history_56_16_relu_2.p) : Pickle file which has training history of our model.
- [fsrcnn_56_16_relu_2.h5](fsrcnn_56_16_relu_2.h5) : Our model.
- [nparray_to_nii.ipynb](nparray_to_nii.ipynb) : This code returns super-resolved numpy array to a medical(.nii.gz) file format.

## Testing the model
1. Open [3DFSRCNN_56_16.ipynb](3DFSRCNN_56_16.ipynb).
2. Go to **"Training >> Importing libraries."** section in that file.
3. Execute following code using [Google Colab](https://colab.research.google.com/) or any suitable alternatives.
> In case of colab use python 3 and GPU in runtime.
```
%tensorflow_version 1.x
from skimage import measure, transform
import matplotlib.pyplot as plt
from keras import backend as K
import nibabel as nib
import numpy as np
import glob
import math
```
> If the libraries are missing, please install it using pip.
4. Go to **"Using the model"** section in that file.
5. Execute all the cells in that section.
```
from keras.models import load_model
model = load_model('3Dfsrcnn.h5', custom_objects={'PSNR': PSNR})
```
> In the the above cell, replace "3Dfsrcnn.h5" with path_to_fsrcnn_56_16_relu_2.h5.
```
img = nib.load(files[69])
img_data = img.get_fdata()
img_data = img_data/(np.amax(img_data))
img_data = img_data * 255
hr = np.array(img_data, dtype='uint8')
lr = hr[::2, ::2, ::2]
sr = np.zeros((256, 320, 320), dtype='uint8')
```
> In the above cell, replace "files\[69]" with the path of scan to be tested **(test_high_res.nii.gz)**.
5. About variables:
- *hr* : 3d np array of input scan 
- *lr* : 3d np array of down-scaled (low-res) input scan
- *sr* : 3d np array of super-resolved scan

## Using the model
> Follow steps 1 to 3 previously explained.
> Let's say a scan name "test_low_res.nii.gz" is to be super-resolved using our model.
4. Execute the following code.
```
from keras.models import load_model
model = load_model(path_to_fsrcnn_56_16_relu_2.h5, custom_objects={'PSNR': PSNR})
```
```
img = nib.load(path_to_test_low_res.nii.gz)
img_data = img.get_fdata()
img_data = img_data/(np.amax(img_data))
img_data = img_data * 255
lr = hr[::2, ::2, ::2]
sr = np.zeros((256, 320, 320), dtype='uint8')
```
```
x1 = 0
y = 0
y1 = 0
z = 0
z1 = 0

for i in range(4):
    y = 0
    y1 = 0
    for j in range(5):
        z = 0
        z1 = 0
        for k in range(5):
            lr0 = np.zeros((1, 32, 32, 32), dtype='uint8')
            lr0[0] = lr[x:x+32, y:y+32, z:z+32]
            lr0.shape = lr0.shape + (1,)
            sr0 = model.predict(lr0)

            sr0 = np.reshape(sr0, (1,64,64,64))
            sr[x1:x1+64, y1:y1+64, z1:z1+64] = sr0[0]

            z = z + 32
            z1 = z1 + 64
        y = y + 32
        y1 = y1 + 64
    x = x + 32
    x1 = x1 + 64
```
> The above cells are assuming the low-res scan dimension is 128x160x160. For other dimension, small changes needs to be done.
5. About variables:
- *lr* : 3d np array of low resolution input scan
- *sr* : 3d np array of super-resolved scan
6. The super-resolved 3d numpy array can be converted to medical file format using [nparray_to_nii.ipynb](nparray_to_nii.ipynb).
