#------

实验四      刘魏鑫宁

#------

**感想**:

这次的实验有坑...

因为我一开始做的是错的..现在才改过来

#------

**思路**:

1.首先这道题考察的其实是一个矩阵对应位置相乘的算法,其实还是很简单的.

​	1>分两步,如果矩阵是3*3,那么要注意,边界如果是空的,那么设为0,也就是说中心点是从(1,1)开始的,因为(0,0)作为中心点,上面是空的,就不能运算了,5* *5同理:

 	

```c++
//如果在边界设为0
				if (N==3&&!(i > 0 && i < imgYlen - 1 && j>0 && j < imgXlen-1))
					buffTmpw[i*imgXlen + j] = (GByte)0;
				else if(N == 5 && !(i >= 2 && i <= imgYlen - 3 && j>=2 && j <= imgXlen-3))
					buffTmpw[i*imgXlen + j] = (GByte)0;
```

2.然后就是中心点是正常的代码,即可以乘起来的那部分:

```c++
double ans = 0;
					//如果阶数为3
					if (N == 3)
					{
						for (int m = 0; m < 3; m++)
						{
							for (int n = 0; n < 3; n++)
							{
								temp3[m][n] = (double)buffTmp[(i-1+m)*imgXlen + j-1+n];
								//全部加起来
								if (order == 1)
									ans += temp3[m][n] * matrix1[m][n];
								if (order == 3)
									ans += temp3[m][n] * matrix3[m][n];
								if (order == 4)
									ans += temp3[m][n] * matrix4[m][n];
								if (order == 5)
									ans += temp3[m][n] * matrix5[m][n];
							}
						}
					}
					//如果阶数为5
					else if (N == 5)
					{
						for (int m = 0; m < 5; m++)
						{
							for (int n = 0; n < 5; n++)
							{
								temp5[m][n] = (double)buffTmp[(i - 2 + m)*imgXlen + j - 2 + n];
								//全部加起来
								if (order == 2)
									ans += temp5[m][n] * matrix2[m][n];
								if (order == 6)
									ans += temp5[m][n] * matrix6[m][n];
							}
						}
					}
```

因为每次下标开始都是从中心点开始的,但是我们遍历给定的卷积核矩阵当然是从(0,0)开始遍历,而不是从中心开始,所以就先把原图像对应的矩阵先赋给一个新的矩阵temp+阶数(命名),注意不同卷积核对应的矩阵不一样.

3.然后你以为这样就可以了嘛~我真是太天真了~~

结果差强人意!图像就像在洗衣机里洗了一遍掉色严重!

但是和同学交流发现要用不同的内存来读和写,即:

```c++
//分配内存
		buffTmp = (GByte*)CPLMalloc(imgXlen*imgYlen * sizeof(GByte));
		buffTmpw = (GByte*)CPLMalloc(imgXlen*imgYlen * sizeof(GByte));
```

不过看其他也有同学是直接改的,并没有用到两个内存区,但是我不用的话会报错.这就是疑问了.

4.加完后的处理,注意浮雕的第五个是要加128的,然后就是不在[0,255]的像素值的处理,简单粗暴:

```c++
if(order==5)
					ans += 128;
					ans *= mul[order - 1];
					if (ans < 0) ans = 0;
					else if (ans > 255) ans = 255;
					buffTmpw[i*imgXlen+j] = (GByte)ans;
```

5.图像展示:

1>均值模糊:

![](http://ww1.sinaimg.cn/large/006y6mwBly1fwy21zpetrj3076075gqe.jpg)

2>运动模糊:



![](http://ww1.sinaimg.cn/large/006y6mwBly1fwy231cxd0j3075077n1r.jpg)

3>边缘检测:

![](http://ww1.sinaimg.cn/large/006y6mwBly1fwy23o3h0ij3075075jut.jpg)

4>锐化:

![](http://ww1.sinaimg.cn/large/006y6mwBly1fwy24b8c5hj307307279l.jpg)

5>浮雕:

![](http://ww1.sinaimg.cn/large/006y6mwBly1fwy24rzp76j307507279a.jpg)

6>高斯模糊:

![](http://ww1.sinaimg.cn/large/006y6mwBly1fwy25kqcp2j3074071wj3.jpg)



**心得**:

这次实验超级有趣!然后有时和同学讨论真的可以得到非常有利的思路,闭门造车总是不好的,如果加上头脑风暴的话可以事半功倍~





