软件工程实验报告

学号：16020031015		姓名：段曦		专业：16计算机科学与技术

实验源代码：

	#include <iostream>
	using namespace std;
	#include "./gdal/gdal_priv.h"
	#pragma comment(lib,"gdal_i.lib")
	
	int main() {
		//输入图像
		GDALDataset* poSrcDS;
		//输出图像
		GDALDataset* poDstDS;
		//图像的宽度和高度
		int imgXlen, imgYlen;
		//输入图像路径
		const char* srcPath = "tree.jpg";
		//输出图像路径
		const char* dstPath = "res.tif";
		//图像内存存储
		GByte* buffTmp;
		//图像波段数
		int i, bandNUm;
	
		//注册驱动
		GDALAllRegister();
	
		//打开图像
		poSrcDS = (GDALDataset*)GDALOpenShared(srcPath, GA_ReadOnly);
	
		//获取图像宽度,高度和波段数量
		imgXlen = poSrcDS->GetRasterXSize();
		imgYlen = poSrcDS->GetRasterYSize();
		bandNUm = poSrcDS->GetRasterCount();
	
		//输出获取的结果
		cout << "Image X Length: " << imgXlen << endl;
		cout << "Image Y Length" << imgYlen << endl;
		cout << "Band Number:" << bandNUm << endl;
	
		//根据图像的宽度和高度分配内存
		buffTmp = (GByte*)CPLMalloc(imgXlen*imgYlen * sizeof(GByte));
	
		//创建输出图像
		poDstDS = GetGDALDriverManager()->GetDriverByName("GTiff")->Create(dstPath, imgXlen, imgYlen, bandNUm, GDT_Byte, NULL);
	
		//一个个波段的输入,然后一个个波段的输出
		for (i = 0; i < bandNUm; i++)
		{
			//这个索引从1开始
			poSrcDS->GetRasterBand(i + 1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmp, imgXlen, imgYlen, GDT_Byte, 0, 0);
			poDstDS->GetRasterBand(i + 1)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmp, imgXlen, imgYlen, GDT_Byte, 0, 0);
			printf("... ... band %d processing ... ...\n", i);
		}
	
		//清除内存
		CPLFree(buffTmp);
		//关闭dataset
		GDALClose(poDstDS);
		GDALClose(poSrcDS);
	
		system("pause");
		return 0;
	}
实验结果

![QQ截图20181008164957](C:\Users\小西\Desktop\QQ截图20181008164957.png)

![QQ截图20181008164856](C:\Users\小西\Desktop\QQ截图20181008164856.png)

遇到的问题：

英文阅读的极度不习惯。