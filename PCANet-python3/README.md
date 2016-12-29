PCANet
------

PCANet is a deep learning network for image classification.  

As the name suggests, weights in the network are calculated by PCA. Because of this characteristics, training of PCANet is extremely fast. Furthermore, class labels are not required in training of PCANet itself.  
Details are described in [the original paper](https://arxiv.org/abs/1404.3606).

## Installation
Just running `python3 setup.py install`.  
If you prefer pip, `pip3 install .` in the PCANet root directory.

## Usage

```python
from pcanet import PCANet

# Arguments are basically passed as tuple in the form (height, width) but int is also allowed. 
# If int is given, the parameter will be converted into (size, size) implicitly.
pcanet = PCANet(
    image_shape=28,  # the size of an input image
    # kernel size, kernel step size, and the number of filters in the first layer, respectively
    filter_shape_l1=2, step_shape_l1=1, n_l1_output=4,
    # kernel size, kernel step size, and the number of filters in the second layer, respectively
    filter_shape_l2=2, step_shape_l2=1, n_l2_output=4,
    block_shape=2  # the size of area to calculate histogram
)

# Check whether all pixels can be considered. Raise ValueError if the structure is not valid.
# Calling this function is optional. PCANet works without this line.
pcanet.validate_structure()

pcanet.fit(images_train)  # Train PCANet

# Trained PCANet behaves as a transformer from images into features.
# `images` is a 3d array in the form (n_images, height, width), who are transformed into feature vectors.
X_train = pcanet.transform(images_train)
X_test = pcanet.transform(images_test) 

# Fit any models you like
from sklearn.ensemble import RandomForestClassifier
model = RandomForestClassifier()
model.fit(X_train, y_train)
y_pred = model.predict(X_test)
```

See [example.py](./example.py) for more details.
