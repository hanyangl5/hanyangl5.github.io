---
layout: post
title: SEU大二机器学习大作业-人脸识别
date: 2020-05-13
---

实验要求

- 将所给人脸照片文件夹作为班级人脸数据库，其中每人有五张图片。
- test文件下存储的是用来识别或认证的不属于数据库中的照片，每人有一张图片。
- 将人脸识别的代码封装成一个函数，命名为facial_recognition，其输入是当前测试图片的路径，且函数要返回最相似的人的名字。
- 将人脸认证的代码封装成一个函数，命名为facial_verification，其输入是数据库中想要比对的人脸id和当前的测试图片的路径，且函数要根据认证的结果，即返回字符串“True”或字符串“False”。
- 定义人名列表，列表中每个元素是人名字符串（即类似于"张三", "李四"），列表名为names，评测中用到的id值即为在names列表中的序号（从0开始）。
- 最终考核结果标准是识别的准确率和时间，以及认证的四项标准。

大作业的名字是人脸识别，但在阅读实验要求后可以看出是一个分类问题


笔者选用resnet50进行分类


    |main.ipynb
    |数据库图片
        |name1
            |name1_1.jpg
            |name1_2.jpg
            ...
        |name2
            |name2_1.jpg
            |name2_2.jpg
            ...
        ...
    |test
        |name1.jpg
        |name2.jpg

首先导入所需要的库以及运行所需要的各项准备：
~~~
import os
import torchvision.models as models
from PIL import Image, ImageDraw
import datetime
from torch.autograd import Variable
import torch
import cv2
import numpy as np
from facenet_pytorch import MTCNN, extract_face
import matplotlib.pyplot as plt
import re
from torchvision import transforms, datasets, models
device = torch.device('cuda:0' if torch.cuda.is_available() else 'cpu')
~~~

构建dataset，因为train和test的目录结构不同，所以需要构建两个dataset

~~~
class TestDataset(torch.utils.data.Dataset):
    def __init__(self, root, list, transform=None):
        self.labellist = list
        self.root = root
        self.transform = transform

    def __len__(self):
        return 1

    def __getitem__(self, index):
        img_path = self.root
        img = Image.open(self.root).convert('RGB')

        label=re.sub("[A-Za-z0-9\!\%\[\]\, \.\-\_\/]", "", img_path)
        label = self.labellist.index(label)
        
        if self.transform:
            img = self.transform(img)
        return img, label


class TrainDataset(torch.utils.data.Dataset):
    def __init__(self, root, list, transform=None):
        self.labellist = list
        self.root = root
        self.transform = transform

        self.images = []
        for files in os.listdir(self.root):
            for filename in os.listdir(os.path.join(root, files)):
                self.images.append(files+'/'+filename)

    def __len__(self):
        return len(self.images)

    def __getitem__(self, index):
        image_index = self.images[index]
        img_path = os.path.join(self.root, image_index)
        img = Image.open(img_path).convert('RGB')
        label = img_path.split('_')[1]
        label = re.sub("[A-Za-z0-9\!\%\[\]\, \.\-\_]", "", label)
        label = self.labellist.index(label)   mapping
        if self.transform:
            img = self.transform(img)
        return img, label
~~~


names = get_name_list('../数据库图片/')

~~~
 transform
data_transform = transforms.Compose([
    transforms.Resize((256, 256)),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                         std=[0.229, 0 .224, 0.225])
])
~~~
加载训练集

~~~
 load train
train_dataset =  TrainDataset(
    root="../数据库图片/",list=names, transform=data_transform)
train_loader = torch.utils.data.DataLoader(
    train_dataset, batch_size=4, shuffle=True)
~~~

##### 初始化模型和训练方法
~~~
net = models.resnet50().to(device)
criterion = torch.nn.CrossEntropyLoss()  # loss function
optimizer = torch.optim.SGD(
    net.parameters(), lr=0.0001, momentum=0.9)  # sgd
epochs = 25
PATH = 'models/resnet152.pth'
if not os.path.exists(PATH):
    print('start training')
    for epoch in range(epochs):  # loop over the dataset multiple times
        running_loss = 0.0
        train_correct = 0
        train_total = 0
        for i, data in enumerate(train_loader, 0):
            inputs, labels = data
            inputs, labels = Variable(
                inputs.cuda()), Variable(labels.cuda())
            optimizer.zero_grad()
            outputs = net(inputs)
            _, train_predicted = torch.max(outputs.data, 1)
            loss = criterion(outputs, labels)
            loss.backward()
            optimizer.step()
            running_loss += loss.item()
            train_total += labels.size(0)
        print('train %d epoch loss: %.3f  ' % (
            epoch + 1, running_loss / train_total))

    print('Finished Training,model saved to ',PATH)  # finish training

    torch.save(net.state_dict(), PATH)
~~~


~~~
def facial_recognition(path):
    test_dataset = Test1Dataset(root=path, list=names, transform=data_transform)
    test_loader = torch.utils.data.DataLoader(test_dataset, batch_size=1, shuffle=False)
    with torch.no_grad():
        for i, data in enumerate(test_loader):
            images, labels = data
            images, labels = images.to(device), labels.to(device)
            outputs = net(images)
            _, predicted = torch.max(outputs.data, 1)
    return names[predicted]


def facial_verification(id,path):
    test_dataset = Test1Dataset(root=path, list=names, transform=data_transform)
    test_loader = torch.utils.data.DataLoader(test_dataset, batch_size=4, shuffle=False)
    dataiter = iter(test_loader)
    images, labels = dataiter.next()
    images, labels = images.to(device), labels.to(device)
    
    outputs = net(images)
    _, predicted = torch.max(outputs, 1)
    print(id,names[predicted[0]])
    return "True" if predicted[0]==id else "False"

~~~
##### 测试
~~~
net = models.resnet50().to(device)
print('load model')
net.load_state_dict(torch.load(PATH))

starttime = datetime.datetime.now()
right = 0
wrong = 0

for item in os.listdir("../test"):

    test_path = "../test/" + item
    if facial_recognition(test_path) == item.split('.')[0]:
        right += 1
    else:
        wrong += 1

accuracy = right / (right+wrong)
endtime = datetime.datetime.now()

print("人脸识别的考察结果：")
print("人脸识别的准确率是:", accuracy)
print("整个人脸识别的运行时间是：", (endtime-starttime).seconds, "s")

tp = 0
tn = 0
fp = 0
fn = 0
'''
for name in names:
    test_path = "../test/" + name + ".jpg"
    for id in range(len(names)):
        result = facial_verification(id, test_path)
        if name == names[id] and result == "True":
            tp +=1
        elif name == names[id] and result == "False":
            fn += 1
        elif name != names[id] and result == "False":
            tn += 1
        else:
            fp += 1

print("人脸认证的考察结果:")
print("精度:", tp/(tp+fp))
print("回归率:", tp/(tp+fn))
print("特异性:", tn/(tn+fp))
print("F1值:", 2*tp/(2*tp+fp+fn))
'''
~~~

