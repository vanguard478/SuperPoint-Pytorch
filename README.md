# SuperPoint-Pytorch (A Pure Pytorch Implementation)
SuperPoint: Self-Supervised Interest Point Detection and Description  


# Thanks  
This work is based on:  
- Tensorflow implementation by [Rémi Pautrat and Paul-Edouard Sarlin](https://github.com/rpautrat/SuperPoint)  
- Official [SuperPointPretrainedNetwork](https://github.com/magicleap/SuperPointPretrainedNetwork).
- [pytorch-superpoint](https://github.com/eric-yyjau/pytorch-superpoint) 
- [Kornia](https://kornia.github.io/)  

# Finished (12/09/2021)
Welcome to star this repository!
# Performance
* Detector repeatibility: **0.67**
* Homography estimation on **images with viewpoint changes** in HPatches dataset: **0.698**  
    - Corresponding result displayed in rpautrat's repository is **0.712**.   
    - Much better performance can be achieved ([0.725](https://github.com/shaofengzeng/SuperPoint-Pytorch/issues/6)) if using the magic-points generated by rpautrat's model. 
    - Another possible way to improve performance is to set more appropriate hyper-parameters, such as `det_threshold`, `nms` and `top_k`.

# New Update (09/04/2021)
* Convert superpoint weight proposed by rpautrat to torch format   
* Usage:
    - 1 Construct network by [superpoint_bn.py](model/superpoint_bn.py) (Refer to [train.py](./train.py) for more details)
    - 2 Set parameter eps=1e-3 for all the BatchNormalization functions in model/modules/cnn/*.py
    - 3 Set parameter momentum=0.01 (**not tested**)
    - 4 Load pretrained weight [superpoint_bn.pth](./superpoint_bn.pth) and run forward propagation
 
 
# Usage
* 0 Update your repository to the latested version (if you have pulled it before)
* 1 Prepare your data. Make directories *data* and *export*. The data directory should look like,
    ```
    data
    |-- coco
    |  |-- train2017
    |  |     |-- a.jpg
    |  |     |-- ...
    |  --- test2017
    |        |-- b.jpg
    |        |-- ...
    |-- hpatches
    |   |-- i_ajuntament
    |   |   |--1.ppm
    |   |   |--...
    |   |   |--H_1_2
    |   |-- ...
    ```
    Create *soft links* if you already have *coco, hpatches* data sets, commands are like,
    ```
    cd data
    ln -s dir_to_coco ./coco
    ```
* 2 Training steps are much similar to [rpautrat/Superpoint](https://github.com/rpautrat/SuperPoint). 
    **However we strongly suggest you read the scripts first before training**
    - 2.0 Modify the following code in train.py, line 61, to save your models, if necessary  
          `if (i%118300==0 and i!=0) or (i+1)==len(dataloader['train']):`  
    - 2.1 set proper epoch in _*.yaml_.
    - 2.2 Train MagicPoint (>1 hours):  
          `python train.py ./config/magic_point_train.yaml`   
          (Note that you have to delete the directory _./data/synthetic_shapes_ whenever you want to regenerate it)
    - 2.3 Export *coco labels data set v1* (>50 hours):   
          `python homo_export_labels.py #using your data dirs`
    - 2.4 Train MagicPoint on *coco labels data set v1* (exported by step 2.2)       
          `python train.py ./config/magic_point_coco_train.py #with correct data dirs` 
    - 2.5 Export *coco labels data set v2* using the magicpoint trained by step 2.3
    - 2.6 Train SuperPoint using *coco labels data set v2* (>12 hours)    
          `python train.py ./config/superpoint_train.py #with correct data dirs`  
    - others. Validate detection repeatability or description  
        ```
        python export_detections_repeatability.py #(very fast)  
        python compute_repeatability.py  #(very fast)
        ## or
        python export_descriptors.py #(> 5.5 hours) 
        python compute_desc_eval.py #(> 1.5 hours)
        ```   
        
    **Descriptions of some important hyper-parameters in YAML files **
    ```
    model
        name: superpoint # train superpoint or magicpoint?
        pretrained_model: None # or path to a pretrained model to load
        using_bn: true # using batch normalization in the model
        det_thresh: 0.001 # point confidence threshold, 1/65
        nms: 4 # nms window size
        topk: -1 # keep top-k points, -1, keep all
     ...
    data:
        name: coco #synthetic
        image_train_path: ['./data/mp_coco_v2/images/train2017',] #several data sets can be list here
        label_train_path: ['./data/mp_coco_v2/labels/train2017/',]
        image_test_path: './data/mp_coco_v2/images/test2017/'
        label_test_path: './data/mp_coco_v2/labels/test2017/'
        ...
        data_dir: './data/hpatches' #path to hpatches dataset
        export_dir: './data/repeatibility/hpatches/sp' #dir where to save output data
    solver:
        model_name: sp #saved model name
    ```

