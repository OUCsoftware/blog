#第三次实验——简单抠像#
实验目的：学会用代码ps，将superman.jpg填充到space.jpg中。

实验步骤：

1.建立基本代码

2.核心代码    

    	poSrcDS1 = (GDALDataset*)GDALOpenShared(srcPath1, GA_ReadOnly);
    	poSrcDS2 = (GDALDataset*)GDALOpenShared(srcPath2, GA_ReadOnly);
    
    	//获取图像，高度和波段数量
    	supXlen = poSrcDS1->GetRasterXSize();
    	supYlen = poSrcDS1->GetRasterYSize();
    	supNum = poSrcDS1->GetRasterCount();//超人
    
    	spaXlen = poSrcDS2->GetRasterXSize();
    	spaYlen = poSrcDS2->GetRasterYSize();
    	spaNum = poSrcDS2->GetRasterCount();//太空
    
    	//创建输出图像
    	poDstDS = GetGDALDriverManager()->GetDriverByName("GTiff")->Create(dstPath, spaXlen, spaYlen, spaNum, GDT_Byte, NULL);
    
    	//根据图像高度和宽度分配内存
    	for (i = 0; i < 3; i++)
    	{
    		supTmp[i] = (GByte*)CPLMalloc(supXlen*supYlen * sizeof(GByte));
    		spaTmp[i] = (GByte*)CPLMalloc(spaXlen*spaYlen * sizeof(GByte));
    	}
    
    	//波段输入然后输出
    
    	buffTmp = (GByte*)CPLMalloc(supXlen*supYlen * sizeof(GByte));
    	
    	//RGB像素读入
    	for (i = 0; i < 3; i++)
    	{
    		poSrcDS1->GetRasterBand(i + 1)->RasterIO(GF_Read, 0, 0, supXlen, supYlen, supTmp[i], supXlen, supYlen, GDT_Byte, 0, 0);
    		poSrcDS2->GetRasterBand(i + 1)->RasterIO(GF_Read, 0, 0, spaXlen, spaYlen, spaTmp[i], spaXlen, spaYlen, GDT_Byte, 0, 0);
    	}
    	for (k = 0; k < 3; k++)
    	{
    	for (j = 0; j < supYlen; j++)
    	{
    		for (i = 0; i < supXlen; i++)
    		{
    			//符合条件即为背景元素，把背景拷贝到目标对象
    			if ((supTmp[0][j*supXlen + i] (GByte)10 && supTmp[0][j*supXlen + i] < (GByte)160) && (supTmp[1][j*supXlen + i] (GByte)100 && supTmp[1][j*supXlen + i] < (GByte)220) && (supTmp[2][j*supXlen + i] (GByte)10 && supTmp[2][j*supXlen + i] < (GByte)160))
    			{
    				buffTmp[j*supXlen + i] = spaTmp[k][j*supXlen + i];
    			}
    			else
    			{
    				buffTmp[j*supXlen + i] = supTmp[k][j*supXlen + i];//覆盖超人
    			}
    		}
    	}
    	
    		poDstDS->GetRasterBand(k + 1)->RasterIO(GF_Write, 0, 0, supXlen, supYlen, buffTmp, supXlen, supYlen, GDT_Byte, 0, 0);
    	}

3.实验结果
![](https://wx3.sinaimg.cn/mw1024/00771gaply1fxmcxfyntfj30hu0ddwlg.jpg)

4.实验中的错误
![](https://wx1.sinaimg.cn/mw1024/00771gaply1fxmcxfomb8j30ht0dgaal.jpg)
    

     for (i = 0; i < supXlen; i++)
    		{
    			//符合条件即为背景元素，把背景拷贝到目标对象
    			if ((supTmp[0][j*supXlen + i] (GByte)10 && supTmp[0][j*supXlen + i] < (GByte)160) && (supTmp[1][j*supXlen + i] (GByte)100 && supTmp[1][j*supXlen + i] < (GByte)220) && (supTmp[2][j*supXlen + i] (GByte)10 && supTmp[2][j*supXlen + i] < (GByte)160))
    			{
    				buffTmp[j*supXlen + i] = spaTmp[0][j*supXlen + i];
    				buffTmp[j*supXlen + i] = spaTmp[1][j*supXlen + i];
    				buffTmp[j*supXlen + i] = spaTmp[2][j*supXlen + i];
    			}
    		}
我将结果所有的点都覆盖成space.jpg中的同一个像素，导致所有像素都呈灰色，于是在最外面又家里了一个for循环，然后就好了。