# Action Recognition with Deep Learning

This branch hosts the code for the technical report ["Towards Good Practices for Very Deep Two-stream ConvNets"](http://arxiv.org/abs/1507.02159), and more.

### Updates
- Dec 23, 2015
  * Refactored cudnn wrapper to control overall memory consumption. Will automatically find the best algorithm combination under memory constraint.
- Dec 17, 2015
  * cuDNN v4 support: faster convolution and batch normalization (around 20% performance gain).
- Nov 22, 2015
  * Now python layer can expose a `prefetch()` method, which will be run in parallel with network processing.

[Full Change Log](CHANGELOG.md)

### Features
- `VideoDataLayer` for inputing video data.
- Training on optical flow data. 
- Data augmentation with fixed corner cropping and multi-scale cropping.
- Parallel training with multiple GPUs.
- cuDNNv3 integration.

### Usage
Generally it's the same as the original caffe. Please see the original README. 
Please see following instruction for accessing features above. More detailed documentation is on the way.

- Video/optic flow data
  - First use the [optical flow extraction tool](https://github.com/wanglimin/dense_flow) to convert videos to RGB images and opitcal flow images.
  - A new data layer called `VideoDataLayer` has been added to support multi-frame input. See the UCF101 sample for how to use it.
  - **Note:** The `VideoDataLayer` can only input the optical-flow images generated by the tool listed above.
- Fixed corner cropping augmentation
  - Set `fix_crop` to `true` in `tranform_param` of network's protocol buffer definition.
- "Multi-scale" cropping augmentation
  - Set `multi_scale` to `true` in `transform_param`
  - In `transform_param`, specify `scale_ratios` as a list of floats smaller than one, default is `[1, .875, .75, .65]`
  - In `transform_param`, specify `max_distort` to an integer, which will limit the aspect ratio distortion, default to `1`
- cuDNN v4
 - The cuDNN v4 wrapper has optimized engines for convolution and batch normalization.
 - The solver protobuf config has a parameter `richness` which specifies the total GPU memory in MBs available to the cudnn convolution engine as workspaces. Default `richness` is 300 (300MB). Using this parameter you can control the GPU memory consumption of training, the system will find the best setup under the memory limit for you.
- Training with multiple GPUs
  - Requires OpenMPI > 1.7.4 ([Why?](https://www.open-mpi.org/faq/?category=runcuda)). **Remember to compile your OpenMPI with option `--with-cuda`**
  - Specify list of GPU IDs to be used for training, in the solver protocol buffer definition, like `device_id: [0,1,2,3]`
  - Compile using cmake and use `mpirun` to launch caffe executable, like 
```bash
mkdir build && cd build
cmake .. -DUSE_MPI=ON
make && make install
mpirun -np 4 ./install/bin/caffe train --solver=<Your Solver File> [--weights=<Pretrained caffemodel>]
```
**Note**: actual batch_size will be `num_device` times `batch_size` specified in network's prototxt.

### Working Examples
- Action recognition on UCF101
  - [Project Site](http://personal.ie.cuhk.edu.hk/~xy012/others/action_recog/)
  - [Caffe Model Files](https://github.com/yjxiong/caffe/tree/action_recog/models/action_recognition)
  - [Training scripts and data files examples](https://github.com/yjxiong/caffe/tree/action_recog/examples/action_recognition)
  - [Optical Flow Data](http://mmlab.siat.ac.cn/very_deep_two_stream_model/ucf101_flow_img_tvl1_gpu.zip)
- Scene recognition on Places205
  - [Model Files](https://github.com/wanglimin/Places205-VGGNet)
  - [Technical Report](http://wanglimin.github.io/papers/WangGHQ15.pdf)

### Extension
Currently all existing data layers sub-classed from `BasePrefetchingDataLayer` support parallel training. If you have newly added layer which is also sub-classed from `BasePrefetchingDataLayer`, simply implement the virtual method 
```C++
inline virtual void advance_cursor();
```
Its function should be forwarding the "data cursor" in your data layer for one step. Then your new layer will be able to provide support for parallel training.

### Questions
Contact 
- [Limin Wang](http://wanglimin.github.io/)
- [Yuanjun Xiong](http://personal.ie.cuhk.edu.hk/~xy012/)

### Citation
You are encouraged to also cite the following report if you find this repo helpful

```
@article{MultiGPUCaffe2015,
  author    = {Limin Wang and
               Yuanjun Xiong and
               Zhe Wang and
               Yu Qiao},
  title     = {Towards Good Practices for Very Deep Two-Stream ConvNets},
  journal   = {CoRR},
  volume    = {abs/1507.02159},
  year      = {2015},
  url       = {http://arxiv.org/abs/1507.02159},
}
```

----
Following is the original README of Caffe.

# Caffe

Caffe is a deep learning framework made with expression, speed, and modularity in mind.
It is developed by the Berkeley Vision and Learning Center ([BVLC](http://bvlc.eecs.berkeley.edu)) and community contributors.

Check out the [project site](http://caffe.berkeleyvision.org) for all the details like

- [DIY Deep Learning for Vision with Caffe](https://docs.google.com/presentation/d/1UeKXVgRvvxg9OUdh_UiC5G71UMscNPlvArsWER41PsU/edit#slide=id.p)
- [Tutorial Documentation](http://caffe.berkeleyvision.org/tutorial/)
- [BVLC reference models](http://caffe.berkeleyvision.org/model_zoo.html) and the [community model zoo](https://github.com/BVLC/caffe/wiki/Model-Zoo)
- [Installation instructions](http://caffe.berkeleyvision.org/installation.html)

and step-by-step examples.

[![Join the chat at https://gitter.im/BVLC/caffe](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/BVLC/caffe?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Please join the [caffe-users group](https://groups.google.com/forum/#!forum/caffe-users) or [gitter chat](https://gitter.im/BVLC/caffe) to ask questions and talk about methods and models.
Framework development discussions and thorough bug reports are collected on [Issues](https://github.com/BVLC/caffe/issues).

Happy brewing!

## License and Citation

Caffe is released under the [BSD 2-Clause license](https://github.com/BVLC/caffe/blob/master/LICENSE).
The BVLC reference models are released for unrestricted use.

Please cite Caffe in your publications if it helps your research:

    @article{jia2014caffe,
      Author = {Jia, Yangqing and Shelhamer, Evan and Donahue, Jeff and Karayev, Sergey and Long, Jonathan and Girshick, Ross and Guadarrama, Sergio and Darrell, Trevor},
      Journal = {arXiv preprint arXiv:1408.5093},
      Title = {Caffe: Convolutional Architecture for Fast Feature Embedding},
      Year = {2014}
    }
