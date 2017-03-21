Title:  Image recognition with Keras, Tensorflow, and InceptionV3
Category: Software
Tags: Python, TensorFlow, Machine Learning, Keras
Author: Trevor
Date: 2017-03-17


Neural networks are a powerful tool for teaching computers to recognize complex patterns, and now tools like [Keras](https://keras.io/) and [TensorFlow](https://www.tensorflow.org/) are beginning to make them a practical tool for programmers who don't have a PhD in machine learning.

One very powerful aspect of these tools is the ability to share pre-trained models with others. There are many tutorials and courses that will walk you through the process of building a neural net and training it on some data set.  But in other areas of software development we are far more likely to use off-the-shelf implementations of common algorithms rather than rolling them ourselves.  We might work through implementing a sort algorithm or a binary tree in order to better understand the concepts, but having done so we almost always end up using the algorithms that come built in to our language or programming environment.

I suspect we'll see the same sort of thing happen in the machine learning world. While being able to train models on our own data will continue to be extremely valuable, there will be many cases where a model already exists that does what we want, and we'll just want to plug it in to our data.

Keras already provides some pre-trained models: in this article, I'll use the [Inception V3](https://www.tensorflow.org/tutorials/image_recognition) model to classify an image.


```python
import numpy as np
import keras
```


```python
from keras.preprocessing import image
from keras.applications.inception_v3 import decode_predictions
from keras.applications.inception_v3 import preprocess_input
```

### Load the pre-trained model


```python
inception=keras.applications.inception_v3.InceptionV3(
    include_top=True, 
    weights='imagenet', 
    input_tensor=None, 
    input_shape=None
)
```

(This actually downloads the weights from github.  Keras saves your model files in `~/.keras/models` in the [HDF5](https://en.wikipedia.org/wiki/Hierarchical_Data_Format) file format.)


```python
!ls  ~/.keras/models
```
    inception_v3_weights_tf_dim_ordering_tf_kernels.h5
    


```python
inception
```




    <keras.engine.training.Model at 0x7f6946e537b8>




```python
inception.summary()
```

    ____________________________________________________________________________________________________
    Layer (type)                     Output Shape          Param #     Connected to                     
    ====================================================================================================
    input_1 (InputLayer)             (None, 299, 299, 3)   0                                            
    ____________________________________________________________________________________________________
    conv2d_1 (Conv2D)                (None, 149, 149, 32)  864                                          
    ____________________________________________________________________________________________________
    batch_normalization_1 (BatchNorm (None, 149, 149, 32)  96                                           
    
    
    (snipped several hundred lines here...)
    
    
    
    mixed10 (Concatenate)            (None, 8, 8, 2048)    0                                            
    ____________________________________________________________________________________________________
    avg_pool (GlobalAveragePooling2D (None, 2048)          0                                            
    ____________________________________________________________________________________________________
    predictions (Dense)              (None, 1000)          2049000                                      
    ====================================================================================================
    Total params: 23,851,784.0
    Trainable params: 23,817,352.0
    Non-trainable params: 34,432.0
    ____________________________________________________________________________________________________


### Now let's load an image and see if Inception can recognize it


```python
img = image.load_img('./hamster.jpg',target_size=(299,299))
```


```python
img
```




![png](images/inception/output_14_0.png)



Keras requires the input data to be in a specific shape.


```python
x = image.img_to_array(img)
x = np.expand_dims(x, axis=0)
x = preprocess_input(x)
```


```python
x.shape
```

    (1, 299, 299, 3)


```python
predictions = inception.predict(x)
prediction  = decode_predictions(predictions)[0][0]
prediction
```




    ('n02342885', 'hamster', 0.91639304)



### And we're done

Inception is pretty confident that this is a picture of a hamster.  Without having 
to do any training ourselves, or really having to know anything at all about neural networks,
we've leveraged a publicly available model to classify our image.

