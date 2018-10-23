## 任务一：把图像指定区域的红色图像置为255

### 1. 任务描述

把输入图像中起始点（100，100），宽度为200像素，高度为150像素的区域红色通道置为255。

### 2. 核心代码

```c++
startX = 100;
startY = 100;
tmpXlen = 200;
tmpYlen = 150;
GDALAllRegister();
poSrcDS = (GDALDataset*)GDALOpenShared(srcPath, GA_ReadOnly);
imgXlen = poSrcDS->GetRasterXSize();
imgYlen = poSrcDS->GetRasterYSize();
bandNum = poSrcDS->GetRasterCount();
buffTmp = (GByte*)CPLMalloc(imgXlen*imgYlen * sizeof(GByte)); //用于加载原图
tmp = (GByte*)CPLMalloc(imgXlen*imgYlen * sizeof(GByte));  //用于加载红色通道的色块
//给目标图分配存储空间
poDstDS = GetGDALDriverManager()->GetDriverByName("GTiff")->Create(dstPath, imgXlen, imgYlen, bandNum, GDT_Byte, NULL);
//读出原图并写在目标地址
for (i = 0; i < bandNum; i++)
{
	poSrcDS->GetRasterBand(i+1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmp, imgXlen, imgYlen, GDT_Byte, 0, 0);
	poDstDS->GetRasterBand(i+1)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmp, imgXlen, imgYlen, GDT_Byte, 0, 0);
}
//读取只有红色通道的色块
poSrcDS->GetRasterBand(1)->RasterIO(GF_Read, startX, startY, tmpXlen, tmpYlen, tmp, tmpXlen, tmpYlen, GDT_Byte, 0, 0);
//把红色通道的值置为255
for (j = 0; j < tmpYlen; j++)
{
	for (i = 0; i < tmpXlen; i++)
	{
		tmp[j*tmpXlen + i] = (GByte)255;
	}
}
//写回
poDstDS->GetRasterBand(1)->RasterIO(GF_Write, startX, startY, tmpXlen, tmpYlen, tmp, tmpXlen, tmpYlen, GDT_Byte, 0, 0);
//关文件
CPLFree(tmp);
CPLFree(buffTmp);
GDALClose(poDstDS);
GDALClose(poSrcDS);
```
### 3.实验结果

![红色色块](https://wx1.sinaimg.cn/square/006zdbpcly1fwg9hgvw76j30xd0o0kjl.jpg)

### 4. 困难和解决办法

（1）第一次读取原图写在目标地址的时候没有写循环，导致整个图只有红色通道，效果很差；

（2）红色色块没有图案，只有红色。

解决方法：把用来读写原图的bufftmp和用来读写红色通道小色块的tmp搞混，没有用tmp加载原图，导致图案不能显示。

## 任务二：把图像指定区域置为白色和黑色

### 1. 任务描述

把输入图像中起始点（300，300），宽度为100像素，高度为50像素的区域置为白色，同时，把输入图像中起始点为（500，500），宽度为50像素，高度为100像素的区域置为黑色。

### 2. 任务分析

主要方法和上面相同，最大的不同就是要把所有通道的数值都改变。

### 3.主要代码

```c++
startX1 = 300;
startY1 = 300;
tmpXlen1 = 100;
tmpYlen1 = 50;

startX2 = 500;
startY2 = 500;
tmpXlen2 = 50;
tmpYlen2 = 100;

//重新打开原图
poSrcDS = (GDALDataset*)GDALOpenShared(srcPath, GA_ReadOnly); 
//bufftmp用于读取原图和把原图写入目的地址
buffTmp = (GByte*)CPLMalloc(imgXlen*imgYlen * sizeof(GByte)); 
//tmp用于操作两个色块
tmp1 = (GByte*)CPLMalloc(imgXlen*imgYlen * sizeof(GByte));
tmp2 = (GByte*)CPLMalloc(imgXlen*imgYlen * sizeof(GByte));
//目标图片
poDstDS = GetGDALDriverManager()->GetDriverByName("GTiff")->Create(dstPath2, imgXlen, imgYlen, bandNum, GDT_Byte, NULL);
//逐个波段读写原图
for (i = 0; i < bandNum; i++)
{
	poSrcDS->GetRasterBand(i + 1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmp, imgXlen, imgYlen, GDT_Byte, 0, 0);
	poDstDS->GetRasterBand(i + 1)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmp, imgXlen, imgYlen, GDT_Byte, 0, 0);
}

//把指定色块变成白色，逐个波段操作
for (k = 0; k < bandNum; k++)
{
	poSrcDS->GetRasterBand(k + 1)->RasterIO(GF_Read, startX1, startY1, tmpXlen1, tmpYlen1, tmp1, tmpXlen1, tmpYlen1, GDT_Byte, 0, 0);
    //逐个像素操作
	for (j = 0; j < tmpYlen1; j++)
	{
		for (i = 0; i < tmpXlen1; i++)
		{
			tmp1[j*tmpXlen1 + i] = (GByte)255;
		}
	}
    //写回
	poDstDS->GetRasterBand(k + 1)->RasterIO(GF_Write, startX1, startY1, tmpXlen1, tmpYlen1, tmp1, tmpXlen1, tmpYlen1, GDT_Byte, 0, 0);
}
//把制定波段变成黑色
for (k = 0; k < bandNum; k++)
{
	poSrcDS->GetRasterBand(k + 1)->RasterIO(GF_Read, startX2, startY2, tmpXlen2, tmpYlen2, tmp2, tmpXlen2, tmpYlen2, GDT_Byte, 0, 0);
	for (j = 0; j < tmpYlen2; j++)
	{
		for (i = 0; i < tmpXlen2; i++)
		{
			tmp2[j*tmpXlen2 + i] = (GByte)0;
		}
	}
	poDstDS->GetRasterBand(k + 1)->RasterIO(GF_Write, startX2, startY2, tmpXlen2, tmpYlen2, tmp2, tmpXlen2, tmpYlen2, GDT_Byte, 0, 0);
}

CPLFree(tmp2);
CPLFree(tmp1);
CPLFree(buffTmp);
GDALClose(poDstDS);
GDALClose(poSrcDS);
```
### 4. 实验结果

![黑白色块](https://wx1.sinaimg.cn/square/006zdbpcly1fwg9joo0t5j30xe0o0kjl.jpg)

### 5. 遇到的困难

（1）变成黑白两色需要把每个色道都改变，开始的时候用的方法是设一个Gbyte类型的数组，后来进行优化，把读取每个色道和更改色道颜色放在一个大循环中进行。

（2）.tif文件直接改后缀生成的.jpg文件不能上传到微博图床

解决办法：截图保存成.jpg文件，再上传