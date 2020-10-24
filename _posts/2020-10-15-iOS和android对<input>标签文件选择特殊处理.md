---
title: 'iOS/android Webview 对<input>标签的特殊处理'
layout: post
comments: true
---
在项目集成中，WebApp需要拍照或者选择相册的图片，一般来说前端都是采用`<input>`标签；这个时候浏览器会触发文件选择器，选择一个文件并且返回。在移动端中，iOS WKWebview默认实现了，一般会弹出相册和拍照2个选项，而在android中，默认是没有反应，需要我们单独去实现。
但是一般选择图片，无法满足一些需求场景，比如需要要求对选择的图片进行编辑 涂鸦等，这个时候需要内置的方法进行一个Hook，我这里给出一个方案供大家参考。

## [WKFileUploadPanel.mm](https://opensource.apple.com/source/WebKit2/WebKit2-7601.1.46.9/UIProcess/ios/forms/WKFileUploadPanel.mm.auto.html) 
`WKFileUploadPanel`是Webkit内部的一个类，看名称就清楚这个类的作用是文件上传相关。我们翻看下它的源码不难发现，他其实就弹出一个Sheet，然后选择【相机】或【相册】，然后获取到图片资源然后处理上传。

```
【sheet 代码】
- (void)_showMediaSourceSelectionSheet
{
    NSString *existingString = [self _photoLibraryButtonLabel];
    NSString *cameraString = [self _cameraButtonLabel];
    NSString *cancelString = WEB_UI_STRING_KEY("Cancel", "Cancel (file upload action sheet)", "File Upload alert sheet button string to cancel");

    _actionSheetController = [UIAlertController alertControllerWithTitle:nil message:nil preferredStyle:UIAlertControllerStyleActionSheet];

    UIAlertAction *cancelAction = [UIAlertAction actionWithTitle:cancelString style:UIAlertActionStyleCancel handler:^(UIAlertAction *){
        [self _cancel];
        // We handled cancel ourselves. Prevent the popover controller delegate from cancelling when the popover dismissed.
        [_presentationPopover setDelegate:nil];
    }];

    UIAlertAction *cameraAction = [UIAlertAction actionWithTitle:cameraString style:UIAlertActionStyleDefault handler:^(UIAlertAction *){
        _usingCamera = YES;
        [self _showPhotoPickerWithSourceType:UIImagePickerControllerSourceTypeCamera];
    }];

    UIAlertAction *photoLibraryAction = [UIAlertAction actionWithTitle:existingString style:UIAlertActionStyleDefault handler:^(UIAlertAction *){
        [self _showPhotoPickerWithSourceType:UIImagePickerControllerSourceTypePhotoLibrary];
    }];

    [_actionSheetController addAction:cancelAction];
    [_actionSheetController addAction:cameraAction];
    [_actionSheetController addAction:photoLibraryAction];

    [self _presentForCurrentInterfaceIdiom:_actionSheetController.get()];
}
```

然后通过imagePicker 代理获取资源，进行内部处理

```
- (void)imagePickerController:(UIImagePickerController *)imagePicker didFinishPickingMediaWithInfo:(NSDictionary *)info
{
    // Sometimes both delegates get called, sometimes just one. Always let the
    // multiple selection delegate handle everything if it will get called.
    if ([self _willMultipleSelectionDelegateBeCalled])
        return;

    [self _dismissDisplayAnimated:YES];

    [self _processMediaInfoDictionaries:[NSArray arrayWithObject:info]
        successBlock:^(NSArray *processedResults, NSString *displayString) {
            ASSERT([processedResults count] == 1);
            _WKFileUploadItem *result = [processedResults objectAtIndex:0];
            dispatch_async(dispatch_get_main_queue(), ^{
                [self _chooseFiles:[NSArray arrayWithObject:result.fileURL] displayString:displayString iconImage:result.displayImage];
            });
        }
        failureBlock:^{
            dispatch_async(dispatch_get_main_queue(), ^{
                [self _cancel];
            });
        }
    ];
}

- (void)imagePickerController:(UIImagePickerController *)imagePicker didFinishPickingMultipleMediaWithInfo:(NSArray *)infos
{
    [self _dismissDisplayAnimated:YES];

    [self _processMediaInfoDictionaries:infos
        successBlock:^(NSArray *processedResults, NSString *displayString) {
            UIImage *iconImage = nil;
            NSMutableArray *fileURLs = [NSMutableArray array];
            for (_WKFileUploadItem *result in processedResults) {
                NSURL *fileURL = result.fileURL;
                if (!fileURL)
                    continue;
                [fileURLs addObject:result.fileURL];
                if (!iconImage)
                    iconImage = result.displayImage;
            }

            dispatch_async(dispatch_get_main_queue(), ^{
                [self _chooseFiles:fileURLs displayString:displayString iconImage:iconImage];
            });
        }
        failureBlock:^{
            dispatch_async(dispatch_get_main_queue(), ^{
                [self _cancel];
            });
        }
    ];
}

```

## 方案
如果我们要通过选择图片进行加工，`hook‘ didFinishPickingMediaWithInfo` 是给不错的选择。

```
void webFileUploadHooker(id self, SEL _cmd, UIImagePickerController *imagePicker, NSDictionary *info) {
    NSLog(@"v_imagePickerController hook:%@",info);
}


- (void)swizzling_wKFileUploadPanel { //如果怕被检查出使用内部类，可以稍微混淆下
    Class panelcls = NSClassFromString(@"WKFileUploadPanel");
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wundeclared-selector"
    class_addMethod(panelcls, @selector(v_imagePickerController:didFinishPickingMediaWithInfo:), (IMP)webFileUploadHooker, "v@:");
    Method ori_Method =  class_getInstanceMethod(panelcls, @selector(imagePickerController:didFinishPickingMediaWithInfo:));
    Method my_Method = class_getInstanceMethod(panelcls, @selector(v_imagePickerController:didFinishPickingMediaWithInfo:));
    method_exchangeImplementations(ori_Method, my_Method);
#pragma clang diagnostic pop
}
```

我们截获的Info信息如下：

```
2020-10-24 14:59:11.130816+0800 wkFileUploadPanelHook[29969:7509917] v_imagePickerController hook:{
    UIImagePickerControllerImageURL = "file:///Users/xxx/Library/Developer/CoreSimulator/Devices/6DCE021D-4F7B-4BD4-8970-EF82056E1AD5/data/Containers/Data/Application/60368027-13AE-400A-B559-393F78951D22/tmp/DC6B534A-E29E-4417-BEFA-5BF424B5D4AE.jpeg";
    UIImagePickerControllerMediaType = "public.image";
    UIImagePickerControllerOriginalImage = "<UIImage:0x60000328a6d0 anonymous {3000, 2002}>";
    UIImagePickerControllerReferenceURL = "assets-library://asset/asset.JPG?id=9F983DBA-EC35-42B8-8773-B597CF782EDD&ext=JPG";
}
```

我们对图片添加一个滤镜，然后返回给原函数
```
void webFileUploadHooker(id self, SEL _cmd, UIImagePickerController *imagePicker, NSDictionary *info) {
    NSLog(@"v_imagePickerController hook:%@",info);
    CIImage *ciImage = [[CIImage alloc] initWithImage:info[UIImagePickerControllerOriginalImage]];
    CIFilter *filter = [CIFilter filterWithName:@"CIPhotoEffectTonal" keysAndValues:kCIInputImageKey, ciImage, nil];
    [filter setDefaults];
    CIContext *context = [CIContext contextWithOptions:nil];
    CIImage *outputImage = [filter outputImage];
    CGImageRef cgImage = [context createCGImage:outputImage fromRect:[outputImage extent]];
    UIImage *image = [UIImage imageWithCGImage:cgImage];
    CGImageRelease(cgImage);
    NSMutableDictionary *newDic = [NSMutableDictionary dictionaryWithDictionary:info];
    newDic[UIImagePickerControllerOriginalImage] = image;
    
    NSData *imgData = UIImageJPEGRepresentation(image, 1);
    NSString *path = NSTemporaryDirectory();
    path = [path stringByAppendingFormat:@"%@.jpeg", [NSUUID UUID].UUIDString];
    NSURL *urlpath = [NSURL fileURLWithPath:path];
    [imgData writeToURL:urlpath atomically:YES];
    newDic[UIImagePickerControllerImageURL] = urlpath;
    
    NSLog(@"v_imagePickerController newInfo:%@",newDic);
    #pragma clang diagnostic push
    #pragma clang diagnostic ignored "-Wundeclared-selector"
    [self performSelector:@selector(v_imagePickerController:didFinishPickingMediaWithInfo:) withObject:imagePicker withObject:newDic];
    #pragma clang diagnostic pop
}
```
 
让我们来看下效果
![Oct-24-2020 15-28-53.gif](https://i.loli.net/2020/10/24/np8s1SoyMzELFh4.gif)

[Demo](https://github.com/Dcell/my-test/tree/master/wkFileUploadPanelHook)
> ⚠️
> 1. 未测试兼容性
> 2. 未测试拍照
> 3. 不保证往后兼容


## android
相反的，andriod的自定义性更高，在H5触发input标签后，android默认未处理对应的事件。你只需要实现android webview `WebChromeClient`，复写`onShowFileChooser`方法。

```
@TargetApi(Build.VERSION_CODES.LOLLIPOP)
public boolean onShowFileChooser(WebView webView, ValueCallback<Uri[]> filePathCallback, FileChooserParams fileChooserParams) {
    //通过ValueCallback 返回对应文件URL即可
}
```
怎么选图片，加工都是完全自定义的。

