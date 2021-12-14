---
title: "iOS GUPImage 硬编 Mp4 H264"
layout: post
comments: true
---
# 前言
先前在android上写实时的视频通讯的时候有一个很大的麻烦，就是如何硬编H264(5.0以后android好像有新的媒体框架，可以直接输出想要的裸码数据）
后来在github上找了一个老外的源码，大概的原理是：

1. 录制一个小视频文件，获取sps,pps属性
2. 将视频录制到本地文件，改为数据流
3. 解析视频数据，输出h264
# IOS硬编码
IOS硬编码和android原理一样。
> 首先可以看下IOS如何录制视频<http://objccn.io/issue-23-1/>
然后监听录制文件，解析数据。

那么需求来了：
   现在很多的弹幕视频，实现原理是客户端拿到弹幕文字，本地添加到播放窗口View。<br/>
如果我要动态的加水印，那可能就比较麻烦了。
> android实现方案：YUV添加水印，然后软编输出。

IOS不急我们先看下这个文章
> Core Image 和视频<http://objccn.io/issue-23-2/>

Apple提供了一套API让我们可以对视频数据进行过滤。<br\>

如果我们将过滤过的视频数据再保存到本地文件，然后解析数据不就可以硬编H264了么？<br\>

这时候强大的GPUImage框架排上用场了(具体使用请自己下载)<br/>

剩下就如何解析视频文件。我这里贴一下我写的代码，可能有很多不妥之处。

<pre><code>-(void)onFileUpdate{
    struct stat s;
    fstat([_inputFile fileDescriptor], &s);
    int cReady = (int)(s.st_size - [_inputFile offsetInFile]);//先获取到可用的文件长度
//    NSLog(@"可用文件长度:%d",cReady);
    if (cReady >  8) {//如果文件长度大于8  前面4个字节是长度，后面4个字节表示内容，比如'mdat'
        NSData* hdr = [_inputFile readDataOfLength:cReady];
        unsigned char* p = (unsigned char*) [hdr bytes];
        int offset = 0;
        BOOL isFoundMDAT = NO;
        while (!(isFoundMDAT = to_host(p+offset) == (unsigned int)('mdat')) && offset< cReady -4) {
             offset++;
        }
        if (isFoundMDAT && offset >= 4) {
            unsigned int lenAtom = to_host(p+(offset-4));//获取Mdat 前面的长度
//            NSLog(@"isFoundMDAT:%d",lenAtom);
            if ((cReady - (offset+4)) >= lenAtom && lenAtom >0) {//缓存的数据够长了
                
                unsigned char* pBuf = (unsigned char*)malloc(4);
                for (int i = 0; i < 3; i++)
                {
                    pBuf[i] = 0x0;
                }
                pBuf[3] = 0x01;
                //添加sps pps
                NSMutableData *spsMutableData =  [[NSMutableData alloc] init];
                [spsMutableData appendBytes:pBuf length:4];
                [spsMutableData appendData:sps];
                
                NSMutableData *ppsMutableData =  [[NSMutableData alloc] init];
                [ppsMutableData appendBytes:pBuf length:4];
                [ppsMutableData appendData:pps];
                
                [self.protocol encoder:spsMutableData];
                [self.protocol encoder:ppsMutableData];
                
                
                [_inputFile seekToFileOffset:[_inputFile offsetInFile] -(cReady-(offset+4+lenAtom))];
                NSData *mdatboxData =  [hdr subdataWithRange:NSMakeRange(offset+4, lenAtom)];
                
                //**** 对每一mdat数据进行解析
                int nalOffset = 0;
                while ([mdatboxData length]>(nalOffset +4)) {
                    NSData *nalDataLen =  [mdatboxData subdataWithRange:NSMakeRange(nalOffset, 4)];
                    nalOffset+=4;
                    unsigned char* nalCharLen = (unsigned char*) [nalDataLen bytes];
                    int nalLen = to_host(nalCharLen);
//                    NSLog(@"nal数据长度:%d",nalLen);
                    if (nalLen == 0x6d6f6f76 || nalLen == 0x6d6f6f66 || nalLen == 0x77696465) {
                        NSLog(@"好像有不好的数据");
                    }else{
                        if ([mdatboxData length] > nalOffset +nalLen) {
                            NSData *nalData =  [mdatboxData subdataWithRange:NSMakeRange(nalOffset, nalLen)];
                            NSMutableData *nalMutableData =  [[NSMutableData alloc] init];
                            [nalMutableData appendBytes:pBuf length:4];
                            [nalMutableData appendData:nalData];
//                             NSLog(@"输出一帧");
                            [self.protocol encoder:nalMutableData];
                        }
                    }
                    nalOffset+=nalLen;
                }
                free(pBuf);
            }else{
                //不够的话，等到够再来
                [_inputFile seekToFileOffset:[_inputFile offsetInFile] - cReady];
            }
        }else{//未找到
            [_inputFile seekToFileOffset:[_inputFile offsetInFile] - cReady];
        }
    }
}</code></pre>


   
   


