android在4.4之后提供了打印照片、html文档、自定义文档的支持。

## 安装mopria print service

android设备连接打印机前需要先安装打印机的服务，类似windows上的驱动，不同型号的需要不同的service，比如三星、HP等。google自带的google could print因为墙所以不好用，好在有mopria print service，只要mopria认证过的打印机就能在android设备上打印图片、网页、文档。
[mopria print service下载](http://pan.baidu.com/s/1hr2DTjQ)

## 远程打印

android develop training中有三个打印的例子，打印照片，html内容、文档。

### 打印照片

打印照片是最简单的，需要用到v4 support library PrintHelper这个类。

````
private void doPhotoPrint() {
  PrintHelper photoPrinter = new PrintHelper(getActivity());
  photoPrinter.setScaleMode(PrintHelper.SCALE_MODE_FIT);
  Bitmap bitmap = BitmapFactory.decodeResource(getResources(),
            R.drawable.droids);
    photoPrinter.printBitmap("droids.jpg - test print", bitmap);
}
````

简单的四句就完成啦打印照片,打印的时候调用这个方法就会弹出打印界面。

PrintHelper类还提供了一些别的方法，

* ```boolean systemSupportsPrint()；判断系统是否支持打印```
* ```printBitmap(String jobName, Uri imageFile, PrintHelper.OnPrintFinishCallback callback)```
* ```printBitmap(String jobName, Bitmap bitmap)```
* ```printBitmap(String jobName, Uri imageFile)```

