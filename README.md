# pix2code2

Pix2code是Tony Beltramelli提出的一个网络，它将不同UI设计图转换为各自的DSL代码。

相关论文在[这儿](https://arxiv.org/abs/1705.07962)，demo代码在[这儿](https://github.com/tonybeltramelli/pix2code)

pix2code2试图通过使用autoencoder来改进该网络。该网络涉及使用卷积神经网络（CNN）和反卷积（deconvolution）（或其转置(transpose)）首先在GUI图像上预训练autoencoder。这两个网络总结如下。

![pix2code_model](https://github.com/fjbriones/pix2code2/blob/master/pix2code_model.PNG)  
pix2code论文中提到的pix2code模型

![pix2code2_model](https://github.com/fjbriones/pix2code2/blob/master/pix2code2_model.PNG)  
pix2code2 计划使用的模型

可以看出，这两个网络之间的唯一区别，代替在pix2code网络内部的卷积网络——在VGGNet之后形成的，在网络外部（但——在autoencoder内进行训练。不是在pix2code网络内优化参数，面是在autoencoder内优化参数，然后将编码器的输出被馈送到pix2code2网络。其背后的基本原理是，（表示通用autoencoder中的潜在向量的）特征映射可能与网络的DSL代码有直接关系，其仅仅是网络的文本表示。当autoencoder试图优化网络的参数时，它学习图像本身的不同特征，而不像原来的pix2code网络那样只是尝试用输出参数优化网络代码。

## 代码:
此处的部分代码由Tony Beltramelli制作，并受其相应许可证的约束，有关详细信息，请参阅Lincense_Pix2code文件。对其中一些进行了修改，以适应所提出的pixc2code2网络。数据集也来自原来的pix2code网络。

到目前为止，此网络已经在web数据集上进行了训练，得到的权重文件也包含在这个git工程中。

## 代码用法:

准备数据：
```sh
# 重组和解压数据
cd datasets
zip -F pix2code_datasets.zip --out datasets.zip
unzip datasets.zip

cd ../model

# 分离训练集数据和评估集数据，确保在评估集数据中没有训练集数据
# 用法: build_datasets.py <input path> <distribution (default: 6)>
./build_datasets.py ../datasets/web/all_data

# 转换在训练数据集中的图片（规范像素值为0~1之间的小数、调整图片尺寸为256x256）为numpy数组（如果你想要上传数据集到云端来训练你的模型，请使用较小的文件）
# 用法: convert_imgs_to_arrays.py <input path> <output path>
./convert_imgs_to_arrays.py ../datasets/web/training_set ../datasets/web/training_features
```
训练模型:
```sh
cd model

# 指定训练数据的输入路径 和 保存训练模型及元数据的输出路径
# 用法: train.py <input path> <output path> <train_autoencoder (default: 0)>
./train.py ../datasets/web/training_set ../bin

# 对已经预处理为数组的图片进行训练
./train.py ../datasets/web/training_features ../bin

# 使用autoencoder进行训练
./train.py ../datasets/web/training_features ../bin 1
```

为Web评估集中的一批GUI生成代码，如果要使用预训练权重，请解压缩此[文件](https://1drv.ms/u/s!Ao8Y5FscWK9imo0GtV5u3sXOr6sc_A) 并将 pix2code2.h5 文件复制到 bin 文件夹：
```sh
mkdir code
cd model

# 生成DSL代码(.gui后缀的文件), 默认搜索方法为 greedy（贪婪）
# 用法: generate.py <trained weights path> <trained model name> <input image> <output path> <search method (default: greedy)>
./generate.py ../bin pix2code2 ../datasets/web/eval_set ../code

# 等同于上面的命令
./generate.py ../bin pix2code2 ../datasets/web/eval_set ../code greedy

# 使用beam搜索并且beam宽度为3，生成 DSL 代码
./generate.py ../bin pix2code2 ../datasets/web/eval_set ../code 3
```

对单张界面图生成代码:
```sh
mkdir code
cd model

# 生成DSL代码(.gui后缀的文件), 默认搜索方法为 greedy（贪婪）
# 用法: sample.py <trained weights path> <trained model name> <input image> <output path> <search method (default: greedy)>
./sample.py ../bin pix2code2 ../test_gui.png ../code

# 等同于上面的命令
./sample.py ../bin pix2code2 ../test_gui.png ../code greedy

# 使用beam搜索并且beam宽度为3，生成 DSL 代码
./sample.py ../bin pix2code2 ../test_gui.png ../code 3
```

编译生成的DSL代码到目标语言:
```sh
cd compiler

# 编译 .gui 文件为 Android XML UI文件
./android-compiler.py <input file path>.gui

# 编译 .gui 文件为 iOS Storyboard文件
./ios-compiler.py <input file path>.gui

# 编译 .gui 文件为 HTML/CSS (基于Bootstrap样式)
./web-compiler.py <input file path>.gui
```

## 测试结果示例
可以在文件夹[tests](https://github.com/fjbriones/pix2code2/tree/master/tests)内找到输入图像和输出html文件示例。图像来自Web图像数据集下的eval_set目录。虽有重构代码，编译器和数据集仍然来自Tony Beltramelli的原作。