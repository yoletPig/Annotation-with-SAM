# 效果演示
![请添加图片描述](https://i-blog.csdnimg.cn/direct/b7f9745ab3544c4096342264553720da.gif)
# 克隆代码
```bash
git clone https://github.com/yoletPig/Annotation-with-SAM.git
```
# 安装SAM

```bash
cd segment-anything
pip install -e .
```
# 安装SAM-Tool依赖包

```bash
pip install -r requirements.txt
```
# 下载权重

```bash
wget https://dl.fbaipublicfiles.com/segment_anything/sam_vit_h_4b8939.pth
```
# 数据集结构
把你的数据放在同一个图片文件夹（dataset_path/images）中，后续会自动处理提取 embeddings 特征并保存为 npy 后缀的文件。
```txt
/dataset_path
    /images
        xxx.jpg/png
    /embeddings
    	xxx.npy
```
# 提取embeddings

```bash
python helpers/extract_embeddings.py --checkpoint-path sam_vit_h_4b8939.pth --dataset-folder <dataset_path> --device cpu/cuda
```
**注意如果是非图片数据会报错，请确定images文件下没有非图片数据**
# 转onnx文件

```bash
python helpers/generate_onnx.py --checkpoint-path sam_vit_h_4b8939.pth --onnx-model-path ./sam_onnx.onnx --orig-im-size 720 1280
```
**我有提供转好的onnx的文件，如果你转换有问题可以尝试我的onnx文件，值得注意的是，opset-version需要15及以上**
**这里不支持动态图片输入，如果需要别的尺寸的需要重新转换onnx文件。**
# 启动可视化工具

```bash
python ../SAM-Tool/segment_anything_annotator.py --onnx-model-path sam_onnx.onnx --dataset-path <dataset_path> --categories cat,dog
```
**类别这里使用英文逗号分割，不带空格**
# 启动可视化可能会遇到的问题

 1、没有配置X11转发
```
qt.qpa.plugin: Could not load the Qt platform plugin "xcb" in "" even though it was found.
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.

Available platform plugins are: eglfs, linuxfb, minimal, minimalegl, offscreen, vnc, wayland-egl, wayland, wayland-xcomposite-egl, wayland-xcomposite-glx, webgl, xcb.

Aborted (core dumped)
```
解决方法参考我之前的文章：[（最新+详细+Pycharm远程调试GUI程序）解决qt.qpa.xcb: could not connect to display问题](https://blog.csdn.net/qq_42840203/article/details/127935439)

2、动态链接库缺失
如果发现配置x11-forwarding后还是上面的报错，需要进行以下步骤来定位问题。
```bash
vim ~/.bashrc
# 在最后一行加上
export QT_DEBUG_PLUGINS=1
source ~/.bashrc
```
上述步骤可以打印详细的报错信息。
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/8c6d1c57ea7e462d8fa1f4acba6bdf69.png)
可以发现是动态链接库缺失，我们进入存放库的路径

```bash
cd /root/miniconda3/lib/python3.8/site-packages/PyQt5/Qt5/plugins/platforms/
# 查看关联内容
ldd libqxcb.so
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/ec4ff14f81f743eca021942d1b537df7.png)
对于缺失的链接库我们安装上即可，比如libxcb-icccm.so.4 对应的库是 libxcb-icccm4，libxkbcommon-x11.so.0对应libxkbcommon-x11-0

```bash
sudo apt-get update
sudo apt-get install libxcb-icccm4 libxcb-keysyms1 libxcb-render-util0 libxcb-xkb1 libxcb-image0 libxcb-keysyms1 libxcb-render-util0 libxcb-xkb1 libxkbcommon-x11-0
```
# 参考资料
[Ubuntu18.04下解决Qt出现qt.qpa.plugin:Could not load the Qt platform plugin “xcb“问题](https://blog.csdn.net/LOVEmy134611/article/details/107212845)
[SAM-Tool](https://github.com/zhouayi/SAM-Tool)
[segment-anything](https://github.com/facebookresearch/segment-anything)