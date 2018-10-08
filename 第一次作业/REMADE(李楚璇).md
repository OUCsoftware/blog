# 第一次实验 ——获取图片的像素
#  实验步骤：
- 1.安装VS
- 2.下载gdal库
- 3.按照老师的步骤进行实验
- 4.在main函数中添加代码
> int main()
> {
> 
> 	GDALDataset* poSrcDS;
> 
> 	GDALDataset* poDstDS;
> 
> 	int imgXlen, imgYlen;
> 
> 	const char* srcPath = "trees.jpg";
> 
> 	const char* dstPath = "res.tif";
> 
> 	GByte* buffTmp;
> 
> 	int i, bandNum;
> 
> 	GDALAllRegister();
> 
> 	poSrcDS = (GDALDataset*)GDALOpenShared(srcPath, GA_ReadOnly);
> 
> 	imgXlen = poSrcDS->GetRasterXSize();
> 
> 	imgYlen = poSrcDS->GetRasterYSize();
> 
> 	bandNum = poSrcDS->GetRasterCount();
> 
> 	cout << "Image X Length:" << imgXlen << endl;
> 	cout << "Image Y Length:" << imgYlen << endl;
> 	cout << "Band number:   " << bandNum << endl;
> 
> 	buffTmp = (GByte*)CPLMalloc(imgXlen*imgYlen * sizeof(GByte));
> 
> 	poDstDS = GetGDALDriverManager()->GetDriverByName("GTiff")->Create(dstPath, imgXlen, imgYlen, bandNum, GDT_Byte, NULL);
> 
> 	for (i = 0; i < bandNum; i++)
> 	{
> 		poSrcDS->GetRasterBand(i + 1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen, buffTmp, imgXlen, imgYlen, GDT_Byte, 0, 0);
> 		poDstDS->GetRasterBand(i + 1)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen, buffTmp, imgXlen, imgYlen, GDT_Byte, 0, 0);
> 		printf("......band %d processing......\n", i);
> 	}
> 
> 	CPLFree(buffTmp);
> 	GDALClose(poDstDS);
> 	GDALClose(poSrcDS);
> 
> 	system("PAUSE");
> 	return 0;
> }
> 因为粗心敲错了几个地方，后来及时发现，并且在VS2017中需要在char*前面需要加上const
> 
> ![](https://wx2.sinaimg.cn/mw1024/00771gaply1fw0xd1altqj308o01qwe9.jpg)
>
> 最后程序的结果如图：
> 
> ![](https://wx2.sinaimg.cn/mw1024/00771gaply1fw0xdeajruj309b048q2w.jpg)
> 
> 与图片属性进行对比
> 
> ![](https://wx1.sinaimg.cn/mw1024/00771gaply1fw0xdeaqdpj305i02eq2q.jpg)
> 
> 发现吻合，则实验成功
> # 实验反思： #
> 因为是初次实验，对新的软件操作不太熟悉，摸索了一段时间。虽然试验成功，但是仍然一知半解，希望在今后的学习中能够自己写代码，独立的完成作业。
> 本次实验我了解了GitHub、markdown、新浪微博图床等之前没有了解的知识，从开始的一无所知到现在的“略懂”，我感到十分的开心。