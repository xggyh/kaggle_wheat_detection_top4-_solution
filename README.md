小麦检测
比赛目的：
(1)小麦数据集密集并且存在大量的重叠，  
(2)野外的照片风会使照片模糊。
(3)此外由于是小麦来自于全球分布广，小麦的品种颜色等方面存在较大的分布差异。
(4)这些因素的影响导致现在所使用的基于yolov3和fasterrcnn的检测器效果达不到理想要求。
比赛细节：比赛有以下几个部分组成：
1数据分析 2 模型选择 3 训练和提交trick策略
数据分析方面：对数据进行分析后，发现小麦密集部分重叠区域较大，应对这一点
(1) 我尝试了mixup，cutmix，cutout和mosaic数据增强。
mixup数据增强方式。处理方式是对每张照片中bounding box的数量和大小进行限制，
对于数量和size小于阈值的照片进行mixup的融合，这样的话可以产生更多的重叠照片，也减少了过多数量的小麦导致模型学到内容过于混乱。
cutmix这里我尝试了将来自不同区域的小麦照片进行拼接，这样模型具有更好的泛化性。
对于cutout中block的面积进行了设置，但是发现越小的block越好，于是就将这一步去除掉了。对于mosaic处理时发现模型精度也产生了下降，后分析重要原因是过多区域的照片进行拼接会导致模型学到内容有所混乱，相同区域拼接时，切割线会破坏小麦的完整性。
(2) 使用albumentations数据增强库进行了大量的数据增强，主要改变了hsv的value，光照数值，添加了高斯噪声等数据增强方法克服原始数据中数据模糊和重叠的问题。
(3) 使用了标签平滑的方式。
2 模型试验：
尝试了多个sota的模型，锁定在了yolov5和efficientdet这两个模型。Yolov5因其输入时的focus结构和良好的cspnet的backbone以及fpn+pan的特征提取层具有最好的测试结果，但是由于license的问题无法在比赛中继续使用，于是我选择了efficientdet，因其不同版本模型大小的灵活性和bifpn的特征融合层也在比赛中具有很好的表现结果。
经过对不同版本的比较发现模型efficientdetd4和efficientdetd5具有最好的实验结果。
3 训练及提交：
（1）对数据进行了5个fold的划分，进行交叉验证。
（2）首先对模型warm，然后选用adamw optimizer搭配余弦退火的学习率更新方式，进行训练。
（3）数据增强选择的是上述的数据增强方式
（4）进行了大量的test time augmentation为了对密集小麦进行检测。
（5）使用wbf（weighted box fusion）方式对测试的bounding box进行融合
使用pseduo label的方法，在测试集中打伪标签并将其放入训练集中进行重新训练
