## 任务： 遥感图像的分块处理  

### 1. 任务描述

​	使用上周学习的IHS方法，对宽幅遥感图像分别使用两种分块方式进行融合：

​	第一种方式：使用256 * 256 大小的块；

​	第二种方式：每块的宽度为图像宽度，高度为256像素。

​	比较两种分块方式的处理效率。

### 2. 核心代码

#### （1）第一种方式：使用256 * 256 大小的块

```C++
//求出行数和列数，以及行列中剩余的小于256的部分
slice = 256;
numx = imgXlen / slice;
numy = imgYlen / slice;
fragx = imgXlen - slice * numx;
fragy = imgYlen - slice * numy;

for (n = 0; n < numy; n++)
	{
		for (m = 0; m < numx; m++)
		{
			poMulDS->GetRasterBand(1)->RasterIO(GF_Read, slice * m, slice * n, slice, slice,
				bandR, slice, slice, GDT_Float32, 0, 0);
			poMulDS->GetRasterBand(2)->RasterIO(GF_Read, slice * m, slice * n, slice, slice,
				bandG, slice, slice, GDT_Float32, 0, 0);
			poMulDS->GetRasterBand(3)->RasterIO(GF_Read, slice * m, slice * n, slice, slice,
				bandB, slice, slice, GDT_Float32, 0, 0);
			poPanDS->GetRasterBand(1)->RasterIO(GF_Read, slice * m, slice * n, slice, slice,
				bandP, slice, slice, GDT_Float32, 0, 0);

			for (i = 0; i < slice * slice; i++)
			{
				bandH[i] = -sqrt(2.0f) / 6.0f*bandR[i] - sqrt(2.0f) / 
                    		6.0f*bandG[i] + sqrt(2.0f) / 3.0f*bandB[i];
				bandS[i] = 1.0f / sqrt(2.0f)*bandR[i] - 1 / sqrt(2.0f)*bandG[i];

				bandR[i] = bandP[i] - 1.0f / sqrt(2.0f)*bandH[i] + 1.0f / sqrt(2.0f)*bandS[i];
				bandG[i] = bandP[i] - 1.0f / sqrt(2.0f)*bandH[i] - 1.0f / sqrt(2.0f)*bandS[i];
				bandB[i] = bandP[i] + sqrt(2.0f)*bandH[i];
			}

			poFusDS->GetRasterBand(1)->RasterIO(GF_Write, slice * m, slice * n, slice, slice,
				bandR, slice, slice, GDT_Float32, 0, 0);
			poFusDS->GetRasterBand(2)->RasterIO(GF_Write, slice * m, slice * n, slice, slice,
				bandG, slice, slice, GDT_Float32, 0, 0);
			poFusDS->GetRasterBand(3)->RasterIO(GF_Write, slice * m, slice * n, slice, slice,
				bandB, slice, slice, GDT_Float32, 0, 0);
		}
		
    	//每行中不足256px宽的块
		if (fragx > 0)
		{
			poMulDS->GetRasterBand(1)->RasterIO(GF_Read, slice * numx, slice * n, fragx, slice,
				bandR, fragx, slice, GDT_Float32, 0, 0);
			poMulDS->GetRasterBand(2)->RasterIO(GF_Read, slice * numx, slice * n, fragx, slice,
				bandG, fragx, slice, GDT_Float32, 0, 0);
			poMulDS->GetRasterBand(3)->RasterIO(GF_Read, slice * numx, slice * n, fragx, slice,
				bandB, fragx, slice, GDT_Float32, 0, 0);
			poPanDS->GetRasterBand(1)->RasterIO(GF_Read, slice * numx, slice * n, fragx, slice,
				bandP, fragx, slice, GDT_Float32, 0, 0);

			for (i = 0; i < slice * (fragx - 1); i++)
			{
				bandH[i] = -sqrt(2.0f) / 6.0f*bandR[i] - sqrt(2.0f) / 
                    		6.0f*bandG[i] + sqrt(2.0f) / 3.0f*bandB[i];
				bandS[i] = 1.0f / sqrt(2.0f)*bandR[i] - 1 / sqrt(2.0f)*bandG[i];

				bandR[i] = bandP[i] - 1.0f / sqrt(2.0f)*bandH[i] + 1.0f / sqrt(2.0f)*bandS[i];
				bandG[i] = bandP[i] - 1.0f / sqrt(2.0f)*bandH[i] - 1.0f / sqrt(2.0f)*bandS[i];
				bandB[i] = bandP[i] + sqrt(2.0f)*bandH[i];
			}

			poFusDS->GetRasterBand(1)->RasterIO(GF_Write, slice * numx, slice * n, fragx, slice,
				bandR, fragx, slice, GDT_Float32, 0, 0);
			poFusDS->GetRasterBand(2)->RasterIO(GF_Write, slice * numx, slice * n, fragx, slice,
				bandG, fragx, slice, GDT_Float32, 0, 0);
			poFusDS->GetRasterBand(3)->RasterIO(GF_Write, slice * numx, slice * n, fragx, slice,
				bandB, fragx, slice, GDT_Float32, 0, 0);
		}
		printf("%d in %d, %d in %d\n", n, numy, m, numx);
	}
	
	//不足256px高的行
	if (fragy > 0)
	{
		for (m = 0; m < numx; m++)
		{
			poMulDS->GetRasterBand(1)->RasterIO(GF_Read, slice * m, slice * numy, slice, fragy,
				bandR, slice, fragy, GDT_Float32, 0, 0);
			poMulDS->GetRasterBand(2)->RasterIO(GF_Read, slice * m, slice * numy, slice, fragy,
				bandG, slice, fragy, GDT_Float32, 0, 0);
			poMulDS->GetRasterBand(3)->RasterIO(GF_Read, slice * m, slice * numy, slice, fragy,
				bandB, slice, fragy, GDT_Float32, 0, 0);
			poPanDS->GetRasterBand(1)->RasterIO(GF_Read, slice * m, slice * numy, slice, fragy,
				bandP, slice, fragy, GDT_Float32, 0, 0);

			for (i = 0; i < slice * (fragy - 1); i++)
			{
				bandH[i] = -sqrt(2.0f) / 6.0f*bandR[i] - sqrt(2.0f) /
                    		6.0f*bandG[i] + sqrt(2.0f) / 3.0f*bandB[i];
				bandS[i] = 1.0f / sqrt(2.0f)*bandR[i] - 1 / sqrt(2.0f)*bandG[i];

				bandR[i] = bandP[i] - 1.0f / sqrt(2.0f)*bandH[i] + 1.0f / sqrt(2.0f)*bandS[i];
				bandG[i] = bandP[i] - 1.0f / sqrt(2.0f)*bandH[i] - 1.0f / sqrt(2.0f)*bandS[i];
				bandB[i] = bandP[i] + sqrt(2.0f)*bandH[i];
			}

			poFusDS->GetRasterBand(1)->RasterIO(GF_Write, slice * m, slice * numy, slice, fragy,
				bandR, slice, fragy, GDT_Float32, 0, 0);
			poFusDS->GetRasterBand(2)->RasterIO(GF_Write, slice * m, slice * numy, slice, fragy,
				bandG, slice, fragy, GDT_Float32, 0, 0);
			poFusDS->GetRasterBand(3)->RasterIO(GF_Write, slice * m, slice * numy, slice, fragy,
				bandB, slice, fragy, GDT_Float32, 0, 0);
		}
		
        //最后一行中不足256px宽的块
		if (fragx > 0)
		{
			poMulDS->GetRasterBand(1)->RasterIO(GF_Read, slice * numx, slice * numy, fragx, 
                                                fragy, bandR, fragx, fragy, GDT_Float32, 0, 0);
			poMulDS->GetRasterBand(2)->RasterIO(GF_Read, slice * numx, slice * numy, fragx, 
                                                fragy, bandG, fragx, fragy, GDT_Float32, 0, 0);
			poMulDS->GetRasterBand(3)->RasterIO(GF_Read, slice * numx, slice * numy, fragx, 
                                                fragy, bandB, fragx, fragy, GDT_Float32, 0, 0);
			poPanDS->GetRasterBand(1)->RasterIO(GF_Read, slice * numx, slice * numy, fragx, 
                                                fragy, bandP, fragx, fragy, GDT_Float32, 0, 0);

			for (i = 0; i < (fragy - 1) * (fragx - 1); i++)
			{
				bandH[i] = -sqrt(2.0f) / 6.0f*bandR[i] - sqrt(2.0f) / 
                    		6.0f*bandG[i] + sqrt(2.0f) / 3.0f*bandB[i];
				bandS[i] = 1.0f / sqrt(2.0f)*bandR[i] - 1 / sqrt(2.0f)*bandG[i];

				bandR[i] = bandP[i] - 1.0f / sqrt(2.0f)*bandH[i] + 1.0f / sqrt(2.0f)*bandS[i];
				bandG[i] = bandP[i] - 1.0f / sqrt(2.0f)*bandH[i] - 1.0f / sqrt(2.0f)*bandS[i];
				bandB[i] = bandP[i] + sqrt(2.0f)*bandH[i];
			}

			poFusDS->GetRasterBand(1)->RasterIO(GF_Write, slice * numx, slice * numy, fragx, 
                                                fragy, bandR, fragx, fragy, GDT_Float32, 0, 0);
			poFusDS->GetRasterBand(2)->RasterIO(GF_Write, slice * numx, slice * numy, fragx, 
                                                fragy, bandG, fragx, fragy, GDT_Float32, 0, 0);
			poFusDS->GetRasterBand(3)->RasterIO(GF_Write, slice * numx, slice * numy, fragx, 
                                                fragy, bandB, fragx, fragy, GDT_Float32, 0, 0);
		}
	}
```



#### （2）第二种方式：每块的宽度为图像宽度，高度为256像素

```c++
    //求出分块数量和剩余行数
	numY = imgYlen / slice;
	left = imgYlen - numY * slice;

	for (j = 0; j < numY; j++)
	{
		poMulDS->GetRasterBand(1)->RasterIO(GF_Read, 0, j*slice, imgXlen, slice,
			bandR, imgXlen, slice, GDT_Float32, 0, 0);
		poMulDS->GetRasterBand(2)->RasterIO(GF_Read, 0, j*slice, imgXlen, slice,
			bandG, imgXlen, slice, GDT_Float32, 0, 0);
		poMulDS->GetRasterBand(3)->RasterIO(GF_Read, 0, j*slice, imgXlen, slice,
			bandB, imgXlen, slice, GDT_Float32, 0, 0);
		poPanDS->GetRasterBand(1)->RasterIO(GF_Read, 0, j*slice, imgXlen, slice,
			bandP, imgXlen, slice, GDT_Float32, 0, 0);

		for (i = 0; i < imgXlen * slice; i++)
		{
			bandH[i] = -sqrt(2.0f) / 6.0f*bandR[i] - sqrt(2.0f) / 
                		6.0f*bandG[i] + sqrt(2.0f) / 3.0f*bandB[i];
			bandS[i] = 1.0f / sqrt(2.0f)*bandR[i] - 1 / sqrt(2.0f)*bandG[i];

			bandR[i] = bandP[i] - 1.0f / sqrt(2.0f)*bandH[i] + 1.0f / sqrt(2.0f)*bandS[i];
			bandG[i] = bandP[i] - 1.0f / sqrt(2.0f)*bandH[i] - 1.0f / sqrt(2.0f)*bandS[i];
			bandB[i] = bandP[i] + sqrt(2.0f)*bandH[i];
		}

		poFusDS->GetRasterBand(1)->RasterIO(GF_Write, 0, j*slice, imgXlen, slice,
			bandR, imgXlen, slice, GDT_Float32, 0, 0);
		poFusDS->GetRasterBand(2)->RasterIO(GF_Write, 0, j*slice, imgXlen, slice,
			bandG, imgXlen, slice, GDT_Float32, 0, 0);
		poFusDS->GetRasterBand(3)->RasterIO(GF_Write, 0, j*slice, imgXlen, slice,
			bandB, imgXlen, slice, GDT_Float32, 0, 0);

		printf("%d in %d\n", j,numY);
	}
	
	//高度小于256px的部分的处理
	if (left > 0)
	{
		poMulDS->GetRasterBand(1)->RasterIO(GF_Read, 0, imgYlen - left, imgXlen, left,
			bandR, imgXlen, left, GDT_Float32, 0, 0);
		poMulDS->GetRasterBand(2)->RasterIO(GF_Read, 0, imgYlen - left, imgXlen, left,
			bandG, imgXlen, left, GDT_Float32, 0, 0);
		poMulDS->GetRasterBand(3)->RasterIO(GF_Read, 0, imgYlen - left, imgXlen, left,
			bandB, imgXlen, left, GDT_Float32, 0, 0);
		poPanDS->GetRasterBand(1)->RasterIO(GF_Read, 0, imgYlen - left, imgXlen, left,
			bandP, imgXlen, left, GDT_Float32, 0, 0);

		for (i = 0; i < imgXlen * (left-1); i++)
		{
			bandH[i] = -sqrt(2.0f) / 6.0f*bandR[i] - sqrt(2.0f) / 
                6.0f*bandG[i] + sqrt(2.0f) / 3.0f*bandB[i];
			bandS[i] = 1.0f / sqrt(2.0f)*bandR[i] - 1 / sqrt(2.0f)*bandG[i];

			bandR[i] = bandP[i] - 1.0f / sqrt(2.0f)*bandH[i] + 1.0f / sqrt(2.0f)*bandS[i];
			bandG[i] = bandP[i] - 1.0f / sqrt(2.0f)*bandH[i] - 1.0f / sqrt(2.0f)*bandS[i];
			bandB[i] = bandP[i] + sqrt(2.0f)*bandH[i];
		}

		poFusDS->GetRasterBand(1)->RasterIO(GF_Write, 0, imgYlen - left-1, imgXlen, left,
			bandR, imgXlen, left, GDT_Float32, 0, 0);
		poFusDS->GetRasterBand(2)->RasterIO(GF_Write, 0, imgYlen - left-1, imgXlen, left,
			bandG, imgXlen, left, GDT_Float32, 0, 0);
		poFusDS->GetRasterBand(3)->RasterIO(GF_Write, 0, imgYlen - left-1, imgXlen, left,
			bandB, imgXlen, left, GDT_Float32, 0, 0);
	}
```

### 3.实验结果

​	两种方法得到的实验结果图相同，如下图所示。由于图片很大，无法在github上传，所以上传到了坚果云，下载地址：https://www.jianguoyun.com/p/DYyghWsQlvujBxiNj5MB

​	这里的效果图做了压缩处理，损失了一些像素，但也能看出效果。

![合成图像](http://ww1.sinaimg.cn/large/006zdbpcly1fyi9ytn6nij31380u0tia.jpg)

与原多光谱图像对比：

![原多光谱图像](http://ww1.sinaimg.cn/large/006zdbpcly1fynv1548tdj30uf0n9q73.jpg)

​	合成的图像明显变清晰了许多。尤其是未压缩前的图像。

​	两种方式得到的图像清晰度相同，但是第二种方法的速度比第一种快近一倍！

### 4. 心得体会

​	实验六和实验五很像，方法基本相同。

​	方法一明显比方法二慢很多，图片过大需要分块处理的时候还是要用方法二。

​	实验做完后很有成就感，把两个巨大的图像进行了融合。这种分块的思想也可以应用于以后的需要大块存储空间、不能直接操作的程序中。