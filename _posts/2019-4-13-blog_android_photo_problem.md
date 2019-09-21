---
layout: post
title: 解决Android 拍照图片被旋转问题
description: 
creditlink: https://blog.csdn.net/qq_27070117/article/details/89290840/
tags: [sample post]
image:
  background: triangular.png
---
<font face="微软雅黑" size="4" style="line-height:32px">

今天在项目中做拍照上传头像相关, 但调用系统相机拍照得到的图片总是旋转90度, 在网上找到了两种答案: </font>

* 第一种如下, 无奈得到的旋转角度总是 0 度 , 无法解决旋转问题

``` java

	//读取图片旋转角度
	public static int readPictureDegree(String path) {
	    int degree = 0;
	    try {
	        ExifInterface exifInterface = new ExifInterface(path);
	        int orientation = exifInterface.getAttributeInt(ExifInterface.TAG_ORIENTATION, ExifInterface.ORIENTATION_NORMAL);
	        LogW.out("readPictureDegree : orientation = " + orientation);
	        if (orientation == ExifInterface.ORIENTATION_ROTATE_90) {
	            degree = 90;
	        } else if (orientation == ExifInterface.ORIENTATION_ROTATE_180) {
	            degree = 180;
	        } else if (orientation == ExifInterface.ORIENTATION_ROTATE_270) {
	            degree = 270;
	        }
	    } catch (IOException e) {
	        e.printStackTrace();
	    }
	    return degree;
	}
	
	//旋转图片
	public static Bitmap rotateBitmap(int angle, Bitmap bitmap) {
	    Matrix matrix = new Matrix();
	    matrix.postRotate(angle);
	    Bitmap rotation = Bitmap.createBitmap(bitmap, 0, 0, bitmap.getWidth(), bitmap.getHeight(),
	            matrix, true);
	    return rotation;
	}

```

* 第二种如下 , 通过比较图片宽高来旋转图片 , 可惜虽然竖着牌的照片能正常显示, 可一旦将相机横向拍摄, 展示到界面就多旋转了 90 度 , 故方法二无效

``` java

	int width = bitmap.getWidth();
	int height = bitmap.getHeight();

	if(width < height) {
	    bitmap = Bitmap.createBitmap(bitmap,0,0,bitmap.getWidth(),bitmap.getWidth());
	}else {
	    Matrix matrix = new Matrix();
	    matrix.postRotate(90);
	    bitmap = Bitmap.createBitmap(bitmap,0,0,bitmap.getHeight(),bitmap.getHeight(),matrix,true);
	}

```

万般无奈之下, 我无意之间使用方法一中的 ExifInterface 打印了拍照后压缩前角度信息, toast 打印结果是 6 , 对应常量 `ExifInterface.ORIENTATION_ROTATE_90` , 可我记得之前获取到的旋转角度是0 , 仔细一想发现获取角度为0那次操作的是压缩后的图片, 我怀疑是不是压缩将图片旋转信息抹掉了. 

于是我在压缩前获取图片旋转信息:

	ExifInterface exifInterface = new ExifInterface(finalUserIcon.getAbsolutePath());//finalUserIcon为压缩前图片
    final int degree = exifInterface.getAttributeInt(ExifInterface.TAG_ORIENTATION, ExifInterface.ORIENTATION_NORMAL);
    UIUtil.debugToast(String.valueOf(degree));

压缩后将原图旋转信息保存替换现有旋转信息:

	try {
        ExifInterface endEI = new ExifInterface(imgPath);//imgPath为压缩后图片路径
        endEI.setAttribute(ExifInterface.TAG_ORIENTATION, String.valueOf(degree));
        endEI.saveAttributes();
    } catch (IOException e) {
        e.printStackTrace();
    }

然后采用方法一的方式获取旋转信息并纠正旋转角度:

	int degree = readPictureDegree(imgPath);
	Bitmap bitmap = rotateBitmap(degree, BitmapFactory.decodeFile(imgPath));
	saveBitmap(imgPath);

		/** 保存方法 */
	public static void saveBitmap(String path, Bitmap bm) {
		File f = new File(path);
		if (f.exists()) {
			f.delete();
		}
		try {
			FileOutputStream out = new FileOutputStream(f);
			bm.compress(Bitmap.CompressFormat.PNG, 90, out);
			out.flush();
			out.close();
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

至此问题得到解决, 无论横着拍照还是竖着拍照都能正确展示.