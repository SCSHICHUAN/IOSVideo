<h1>介绍</h1>
<h4>这个demo是利用AVFoundation来写的，功能是个视频添加水印，文字，动画，旋转视频</h4>
<h1>注意</h1>
<h4>1.这是一个Pod的项目，你如果要直接运行这个demo请 同步一下Pod</h4>
<h4>2.请使用真机运行</h4>


<h1>如果你不用pod</h1>
<h4>你可以新建一个项目，把 “ViewController”里的代码复制到你新建的工程中，然后把红色的删除，当然会有部分的功能缺失</h4>

<h1>Org Code</h1>
<pre>
  #import "ViewController.h"
#import <AVFoundation/AVFoundation.h>
#import <AVKit/AVKit.h>
#import <GPUImage/GPUImage.h>
#import <lottie-ios/Lottie/Lottie.h>


#import "MBProgressHUD.h"

@interface ViewController ()
{
    MBProgressHUD *hud;
}
@property(strong,nonatomic)AVPlayerViewController *player;
@property(strong,nonatomic)NSURL *url;

@end

@implementation ViewController

-(NSURL *)url
{
    if (_url == nil) {
        _url = [[NSURL alloc] initFileURLWithPath:[NSTemporaryDirectory() stringByAppendingPathComponent:@"tmpMov.mov"]];
        NSLog(@"url= [ %@ ]",_url);
    }
    return _url;
}
- (void)viewDidLoad {
    [super viewDidLoad];
}

- (IBAction)water:(UIButton *)sender {
    [self deleteTmp];
    [self mbLoging];
    [self addWatermark];
}
- (IBAction)text:(UIButton *)sender {
    [self deleteTmp];
    [self mbLoging];
    [self addTextTitle];
}
- (IBAction)GIF:(UIButton *)sender {
    [self deleteTmp];
    [self mbLoging];
    [self addGIF];
}
- (IBAction)pngAnimation:(UIButton *)sender {
    [self deleteTmp];
    [self mbLoging];
    [self addPngAnimation];
}
- (IBAction)addAnimation:(UIButton *)sender {
    [self deleteTmp];
    [self mbLoging];
    [self addCAAnimation];
}
- (IBAction)rotationVideo:(UIButton *)sender {
    [self deleteTmp];
    [self mbLoging];
    [self rotationVideo];
}
-(void)mbLoging
{
    hud = [MBProgressHUD showHUDAddedTo:self.view animated:true];
    hud.mode = MBProgressHUDModeIndeterminate;
    hud.label.text = @"制作中...";
}
-(void)deleteTmp
{
    NSString *path = [NSTemporaryDirectory() stringByAppendingPathComponent:@"tmpMov.mov"];
    NSFileManager *fileManager = [NSFileManager defaultManager];
    BOOL isDelete = [fileManager removeItemAtPath:path error:nil];
    if (isDelete) {
        NSLog(@"delete successed");
    }else{
        NSLog(@"delete filed");
    }
}


-(void)playVideo
{
    self.player = [[AVPlayerViewController alloc] init];
    self.player.player = [[AVPlayer alloc] initWithURL:self.url];
    self.player.view.frame = CGRectMake(0, 0, [UIScreen mainScreen].bounds.size.width, [UIScreen mainScreen].bounds.size.height);
    [self.player.player play];
}


-(void)applyVideoEffectsToComposition:(AVMutableVideoComposition *)composition size:(CGSize)size{
    // 1 - set up the overlay
    CALayer *overlayLayer = [CALayer layer];
    UIImage *overlayImage  = [UIImage imageNamed:@"p"];
    
    [overlayLayer setContents:(id)[overlayImage CGImage]];
    overlayLayer.frame = CGRectMake(size.width - 146, 24, 135, 18);
    [overlayLayer setMasksToBounds:YES];
    
    // 2 - set up the parent layer
    CALayer *parentLayer = [CALayer layer];
    CALayer *videoLayer = [CALayer layer];
    parentLayer.frame = CGRectMake(0, 0, size.width, size.height);
    videoLayer.frame = CGRectMake(0, 0, size.width, size.height);
    [parentLayer addSublayer:videoLayer];
    [parentLayer addSublayer:overlayLayer];
    
    //*********** For A Special Time
    CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"opacity"];
    [animation setDuration:0];
    [animation setFromValue:[NSNumber numberWithFloat:1.0]];
    [animation setToValue:[NSNumber numberWithFloat:0.0]];
    [animation setBeginTime:5];
    [animation setRemovedOnCompletion:NO];
    [animation setFillMode:kCAFillModeForwards];
    [overlayLayer addAnimation:animation forKey:@"animateOpacity"];
    
    // 3 - apply magic
    composition.animationTool = [AVVideoCompositionCoreAnimationTool
                                 videoCompositionCoreAnimationToolWithPostProcessingAsVideoLayer:videoLayer inLayer:parentLayer];
}


//视频添加水印
-(void)addWatermark
{
    //创建工作台
    AVMutableComposition *workbench = [AVMutableComposition composition];
    //创建主视频和音频轨道添加到工作台
    AVMutableCompositionTrack *mainVideoTrack = [workbench addMutableTrackWithMediaType:AVMediaTypeVideo preferredTrackID:kCMPersistentTrackID_Invalid];
    AVMutableCompositionTrack *mainAudioTrack = [workbench addMutableTrackWithMediaType:AVMediaTypeAudio preferredTrackID:kCMPersistentTrackID_Invalid];
    //引入外部多媒体 创建资源轨道
    AVURLAsset* videoAsset = [[AVURLAsset alloc]initWithURL:[[NSBundle mainBundle] URLForResource:@"v1" withExtension:@"mp4"] options:nil];
    AVAssetTrack *videoTrack = [[videoAsset tracksWithMediaType:AVMediaTypeVideo] objectAtIndex:0];
    AVAssetTrack *audioTrack = [[videoAsset tracksWithMediaType:AVMediaTypeAudio] objectAtIndex:0];
    
    //向音视频轨道插入资源
    [mainVideoTrack insertTimeRange:CMTimeRangeMake(kCMTimeZero, videoAsset.duration) ofTrack:videoTrack atTime:kCMTimeZero error:nil];
    [mainAudioTrack insertTimeRange:CMTimeRangeMake(kCMTimeZero, videoAsset.duration) ofTrack:audioTrack atTime:kCMTimeZero error:nil];
    
    //获取视频资源的尺寸
    CGSize videoSize = [videoTrack naturalSize];
    
    UIImage *myImage=[UIImage imageNamed:@"p.png"];
    CALayer *imgLay = [CALayer layer];
    imgLay.contents = (__bridge id _Nullable)(myImage.CGImage);
    imgLay.frame = CGRectMake(100, 100, 200, 200);
    imgLay.opacity = 1.0;
    
    //创建父layer
    CALayer *parentLayer = [CALayer layer];
    parentLayer.frame=CGRectMake(0, 0, videoSize.width, videoSize.height);;
    //准备layer为参数，这个决定视频的大小
    CALayer *videoLayer=[CALayer layer];
    videoLayer.frame=CGRectMake(0, 0, videoSize.width, videoSize.height);
    [parentLayer addSublayer:videoLayer];
    [parentLayer addSublayer:imgLay];
    //反转坐标
    parentLayer.geometryFlipped = true;
    
    //创建视频工作台
    AVMutableVideoComposition *videoWorkbench=[AVMutableVideoComposition videoComposition];
    //设置每秒播放的帧数
    videoWorkbench.frameDuration=CMTimeMake(1, 30);
    videoWorkbench.renderSize = videoSize;
    //设置视频层为“videoLayer”，设置父视层为“parentLayer”
    videoWorkbench.animationTool=[AVVideoCompositionCoreAnimationTool videoCompositionCoreAnimationToolWithPostProcessingAsVideoLayer:videoLayer inLayer:parentLayer];
    
    
    //创建视频指令
    AVMutableVideoCompositionInstruction *videoinstruction = [AVMutableVideoCompositionInstruction videoCompositionInstruction];
    //设置时间
    videoinstruction.timeRange = CMTimeRangeMake(kCMTimeZero, [workbench duration]);
    //工作台中的媒体资源创建轨道
    AVAssetTrack *workbenchVideoTrack = [[workbench tracksWithMediaType:AVMediaTypeVideo] objectAtIndex:0];
    //创建工作台视频指令层
    AVMutableVideoCompositionLayerInstruction* layerInstruction = [AVMutableVideoCompositionLayerInstruction videoCompositionLayerInstructionWithAssetTrack:workbenchVideoTrack];
    //视频指令中加入主工作台的资源
    videoinstruction.layerInstructions = [NSArray arrayWithObject:layerInstruction];
    //添加主工作台到“videoWorkbench”工作台中
    videoWorkbench.instructions = [NSArray arrayWithObject: videoinstruction];
    
    
    
    AVAssetExportSession *exportSession = [[AVAssetExportSession alloc] initWithAsset:workbench presetName:AVAssetExportPresetMediumQuality];
    exportSession.videoComposition=videoWorkbench;
    
    exportSession.outputURL = self.url;
    exportSession.outputFileType = AVFileTypeQuickTimeMovie;
    [exportSession exportAsynchronouslyWithCompletionHandler:^{
        switch (exportSession.status) {
            case AVAssetExportSessionStatusUnknown:
                NSLog(@"AVAssetExportSessionStatusUnknown");
                break;
            case AVAssetExportSessionStatusWaiting:
                NSLog(@"AVAssetExportSessionStatusWaiting");
                break;
            case AVAssetExportSessionStatusExporting:
                NSLog(@"AVAssetExportSessionStatusExporting");
                break;
            case AVAssetExportSessionStatusFailed:
                NSLog(@"AVAssetExportSessionStatusFailed");
                break;
            case AVAssetExportSessionStatusCancelled:
                NSLog(@"AVAssetExportSessionStatusCancelled");
                break;
            case AVAssetExportSessionStatusCompleted:
                NSLog(@"export succeed");
                dispatch_async(dispatch_get_main_queue(), ^{
                    [self playVideo];
                    [self presentViewController:self.player animated:YES completion:^{
                        [self->hud hideAnimated:true];
                    }];
                });
                break;
        }
    }];
}
//视频添加文字
-(void)addTextTitle
{
    //创建工作台
    AVMutableComposition *workbench = [AVMutableComposition composition];
    //创建主视频和音频轨道添加到工作台
    AVMutableCompositionTrack *mainVideoTrack = [workbench addMutableTrackWithMediaType:AVMediaTypeVideo preferredTrackID:kCMPersistentTrackID_Invalid];
    AVMutableCompositionTrack *mainAudioTrack = [workbench addMutableTrackWithMediaType:AVMediaTypeAudio preferredTrackID:kCMPersistentTrackID_Invalid];
    //引入外部多媒体 创建资源轨道
    AVURLAsset* videoAsset = [[AVURLAsset alloc]initWithURL:[[NSBundle mainBundle] URLForResource:@"v1" withExtension:@"mp4"] options:nil];
    AVAssetTrack *videoTrack = [[videoAsset tracksWithMediaType:AVMediaTypeVideo] objectAtIndex:0];
    AVAssetTrack *audioTrack = [[videoAsset tracksWithMediaType:AVMediaTypeAudio] objectAtIndex:0];
    
    //向音视频轨道插入资源
    [mainVideoTrack insertTimeRange:CMTimeRangeMake(kCMTimeZero, videoAsset.duration) ofTrack:videoTrack atTime:kCMTimeZero error:nil];
    [mainAudioTrack insertTimeRange:CMTimeRangeMake(kCMTimeZero, videoAsset.duration) ofTrack:audioTrack atTime:kCMTimeZero error:nil];
    
    //获取视频资源的尺寸
    CGSize videoSize = [videoTrack naturalSize];
    
    UIFont *font = [UIFont systemFontOfSize:70.0];
    CATextLayer *tLayer = [[CATextLayer alloc] init];
    [tLayer setFontSize:70];
    [tLayer setString:@"Hello world"];
    [tLayer setAlignmentMode:kCAAlignmentCenter];
    [tLayer setForegroundColor:[[UIColor greenColor] CGColor]];
    [tLayer setBackgroundColor:[UIColor clearColor].CGColor];
    CGSize textSize = [@"Hello world" sizeWithAttributes:[NSDictionary dictionaryWithObjectsAndKeys:font,NSFontAttributeName, nil]];
    [tLayer setFrame:CGRectMake(10, 200, textSize.width+20, textSize.height+10)];
    tLayer.anchorPoint = CGPointMake(0.5, 1.0);
    
    //创建父layer
    CALayer *parentLayer = [CALayer layer];
    parentLayer.frame=CGRectMake(0, 0, videoSize.width, videoSize.height);;
    //准备layer为参数，这个决定视频的大小
    CALayer *videoLayer=[CALayer layer];
    videoLayer.frame=CGRectMake(0, 0, videoSize.width, videoSize.height);
    [parentLayer addSublayer:videoLayer];
    [parentLayer addSublayer:tLayer];
    //反转坐标
    parentLayer.geometryFlipped = true;
    
    //创建视频工作台
    AVMutableVideoComposition *videoWorkbench=[AVMutableVideoComposition videoComposition];
    //设置每秒播放的帧数
    videoWorkbench.frameDuration=CMTimeMake(1, 30);
    videoWorkbench.renderSize = videoSize;
    //设置视频层为“videoLayer”，设置父视层为“parentLayer”
    videoWorkbench.animationTool=[AVVideoCompositionCoreAnimationTool videoCompositionCoreAnimationToolWithPostProcessingAsVideoLayer:videoLayer inLayer:parentLayer];
    
    
    //创建视频指令
    AVMutableVideoCompositionInstruction *videoinstruction = [AVMutableVideoCompositionInstruction videoCompositionInstruction];
    //设置时间
    videoinstruction.timeRange = CMTimeRangeMake(kCMTimeZero, [workbench duration]);
    //工作台中的媒体资源创建轨道
    AVAssetTrack *workbenchVideoTrack = [[workbench tracksWithMediaType:AVMediaTypeVideo] objectAtIndex:0];
    //创建工作台视频指令层
    AVMutableVideoCompositionLayerInstruction* layerInstruction = [AVMutableVideoCompositionLayerInstruction videoCompositionLayerInstructionWithAssetTrack:workbenchVideoTrack];
    //视频指令中加入主工作台的资源
    videoinstruction.layerInstructions = [NSArray arrayWithObject:layerInstruction];
    //添加主工作台到“videoWorkbench”工作台中
    videoWorkbench.instructions = [NSArray arrayWithObject: videoinstruction];
    
    
    
    AVAssetExportSession *exportSession = [[AVAssetExportSession alloc] initWithAsset:workbench presetName:AVAssetExportPresetMediumQuality];
    exportSession.videoComposition=videoWorkbench;
    
    exportSession.outputURL = self.url;
    exportSession.outputFileType = AVFileTypeMPEG4;
    exportSession.shouldOptimizeForNetworkUse = YES;
    [exportSession exportAsynchronouslyWithCompletionHandler:^{
        switch (exportSession.status) {
            case AVAssetExportSessionStatusUnknown:
                NSLog(@"AVAssetExportSessionStatusUnknown");
                break;
            case AVAssetExportSessionStatusWaiting:
                NSLog(@"AVAssetExportSessionStatusWaiting");
                break;
            case AVAssetExportSessionStatusExporting:
                NSLog(@"AVAssetExportSessionStatusExporting");
                break;
            case AVAssetExportSessionStatusFailed:
                NSLog(@"AVAssetExportSessionStatusFailed");
                break;
            case AVAssetExportSessionStatusCancelled:
                NSLog(@"AVAssetExportSessionStatusCancelled");
                break;
            case AVAssetExportSessionStatusCompleted:
                NSLog(@"export succeed");
                dispatch_async(dispatch_get_main_queue(), ^{
                    
                    [self playVideo];
                    [self presentViewController:self.player animated:YES completion:^{
                        [self->hud hideAnimated:true];
                    }];
                });
                break;
        }
    }];
}
//视频添加动画
-(void)addCAAnimation
{
    //创建工作台
    AVMutableComposition *workbench = [AVMutableComposition composition];
    //创建主视频和音频轨道添加到工作台
    AVMutableCompositionTrack *mainVideoTrack = [workbench addMutableTrackWithMediaType:AVMediaTypeVideo preferredTrackID:kCMPersistentTrackID_Invalid];
    AVMutableCompositionTrack *mainAudioTrack = [workbench addMutableTrackWithMediaType:AVMediaTypeAudio preferredTrackID:kCMPersistentTrackID_Invalid];
    //引入外部多媒体 创建资源轨道
    AVURLAsset* videoAsset = [[AVURLAsset alloc]initWithURL:[[NSBundle mainBundle] URLForResource:@"v1" withExtension:@"mp4"] options:nil];
    AVAssetTrack *videoTrack = [[videoAsset tracksWithMediaType:AVMediaTypeVideo] objectAtIndex:0];
    AVAssetTrack *audioTrack = [[videoAsset tracksWithMediaType:AVMediaTypeAudio] objectAtIndex:0];
    
    //向音视频轨道插入资源
    [mainVideoTrack insertTimeRange:CMTimeRangeMake(kCMTimeZero, videoAsset.duration) ofTrack:videoTrack atTime:kCMTimeZero error:nil];
    [mainAudioTrack insertTimeRange:CMTimeRangeMake(kCMTimeZero, videoAsset.duration) ofTrack:audioTrack atTime:kCMTimeZero error:nil];
    
    //获取视频资源的尺寸
    CGSize videoSize = [videoTrack naturalSize];
    
    
    
    //创建父layer
    CALayer *parentLayer = [CALayer layer];
    parentLayer.frame=CGRectMake(0, 0, videoSize.width, videoSize.height);
    //准备layer为参数，这个决定视频的大小
    CALayer *videoLayer=[CALayer layer];
    videoLayer.frame=CGRectMake(0, 0, videoSize.width, videoSize.height);
    [parentLayer addSublayer:videoLayer];
    
    //反转坐标
    parentLayer.geometryFlipped = true;
    
    CALayer *animationLay = [CALayer layer];
    animationLay.frame = CGRectMake(100, 100, 100, 100);
    animationLay.backgroundColor = UIColor.redColor.CGColor;
    animationLay.anchorPoint = CGPointMake(0.5, 1.0);
    
    CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"transform.scale.y"];
    animation.fromValue = @(0.2f);
    animation.toValue = @(-3.0f);
    animation.beginTime = AVCoreAnimationBeginTimeAtZero;
    animation.duration = 2.0f;
    animation.repeatCount = HUGE_VALF;
    animation.removedOnCompletion = NO;
    [animationLay addAnimation:animation forKey:nil];
    
    [parentLayer addSublayer:animationLay];
    
    
    
    
    //创建视频工作台
    AVMutableVideoComposition *videoWorkbench=[AVMutableVideoComposition videoComposition];
    //设置每秒播放的帧数
    videoWorkbench.frameDuration=CMTimeMake(1, 30);
    videoWorkbench.renderSize = videoSize;
    //设置视频层为“videoLayer”，设置父视层为“parentLayer”
    videoWorkbench.animationTool=[AVVideoCompositionCoreAnimationTool videoCompositionCoreAnimationToolWithPostProcessingAsVideoLayer:videoLayer inLayer:parentLayer];
    
    
    //创建视频指令
    AVMutableVideoCompositionInstruction *videoinstruction = [AVMutableVideoCompositionInstruction videoCompositionInstruction];
    //设置时间
    videoinstruction.timeRange = CMTimeRangeMake(kCMTimeZero, [workbench duration]);
    //工作台中的媒体资源创建轨道
    AVAssetTrack *workbenchVideoTrack = [[workbench tracksWithMediaType:AVMediaTypeVideo] objectAtIndex:0];
    //创建工作台视频指令层
    AVMutableVideoCompositionLayerInstruction* layerInstruction = [AVMutableVideoCompositionLayerInstruction videoCompositionLayerInstructionWithAssetTrack:workbenchVideoTrack];
    //视频指令中加入主工作台的资源
    videoinstruction.layerInstructions = [NSArray arrayWithObject:layerInstruction];
    //添加主工作台到“videoWorkbench”工作台中
    videoWorkbench.instructions = [NSArray arrayWithObject: videoinstruction];
    
    
    
    AVAssetExportSession *exportSession = [[AVAssetExportSession alloc] initWithAsset:workbench presetName:AVAssetExportPresetHighestQuality];
    exportSession.videoComposition=videoWorkbench;
    exportSession.outputURL = self.url;
    exportSession.outputFileType = AVFileTypeQuickTimeMovie;
    [exportSession exportAsynchronouslyWithCompletionHandler:^{
        switch (exportSession.status) {
            case AVAssetExportSessionStatusUnknown:
                NSLog(@"AVAssetExportSessionStatusUnknown");
                break;
            case AVAssetExportSessionStatusWaiting:
                NSLog(@"AVAssetExportSessionStatusWaiting");
                break;
            case AVAssetExportSessionStatusExporting:
                NSLog(@"AVAssetExportSessionStatusExporting");
                break;
            case AVAssetExportSessionStatusFailed:
                NSLog(@"AVAssetExportSessionStatusFailed");
                break;
            case AVAssetExportSessionStatusCancelled:
                NSLog(@"AVAssetExportSessionStatusCancelled");
                break;
            case AVAssetExportSessionStatusCompleted:
                NSLog(@"export succeed");
                dispatch_async(dispatch_get_main_queue(), ^{
                    
                    [self playVideo];
                    [self presentViewController:self.player animated:YES completion:^{
                        [self->hud hideAnimated:true];
                    }];
                });
                break;
        }
    }];
}
//视频旋转
-(void)rotationVideo
{
    
    
    
    //创建工作台
    AVMutableComposition *workbench = [AVMutableComposition composition];
    //创建主视频和音频轨道添加到工作台
    AVMutableCompositionTrack *mainVideoTrack = [workbench addMutableTrackWithMediaType:AVMediaTypeVideo preferredTrackID:kCMPersistentTrackID_Invalid];
    AVMutableCompositionTrack *mainAudioTrack = [workbench addMutableTrackWithMediaType:AVMediaTypeAudio preferredTrackID:kCMPersistentTrackID_Invalid];
    //引入外部多媒体 创建资源轨道
    AVURLAsset* videoAsset = [[AVURLAsset alloc]initWithURL:[[NSBundle mainBundle] URLForResource:@"v1" withExtension:@"MOV"] options:nil];
    AVAssetTrack *videoTrack = [[videoAsset tracksWithMediaType:AVMediaTypeVideo] objectAtIndex:0];
    AVAssetTrack *audioTrack = [[videoAsset tracksWithMediaType:AVMediaTypeAudio] objectAtIndex:0];
    
    //向音视频轨道插入资源
    [mainVideoTrack insertTimeRange:CMTimeRangeMake(kCMTimeZero, videoAsset.duration) ofTrack:videoTrack atTime:kCMTimeZero error:nil];
    [mainAudioTrack insertTimeRange:CMTimeRangeMake(kCMTimeZero, videoAsset.duration) ofTrack:audioTrack atTime:kCMTimeZero error:nil];
    
    //设置视频的尺寸，720*1280是竖屏的和横屏的分水岭
    CGSize videoSize = CGSizeMake(720, 1280);
    
    
    
    
    //创建父layer
    CALayer *parentLayer = [CALayer layer];
    parentLayer.backgroundColor = UIColor.redColor.CGColor;
    parentLayer.frame=CGRectMake(0, 0, videoSize.width, videoSize.height);
    //准备layer为参数，这个决定视频的大小
    CALayer *videoLayer=[CALayer layer];
    videoLayer.frame=CGRectMake(0, 0,videoSize.height,videoSize.width);
    videoLayer.backgroundColor = UIColor.yellowColor.CGColor;
    [parentLayer addSublayer:videoLayer];
    
    
    //BA动画 旋转视频，和移动视频
    videoLayer.anchorPoint = CGPointMake(0.5, 0.5);
    //    CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"transform.rotation.z"];
    //    animation.fromValue = @(0.2f);
    //    animation.toValue = [NSNumber numberWithFloat:-M_PI/2];
    //    animation.beginTime = AVCoreAnimationBeginTimeAtZero;
    //    animation.duration = 0.000f;
    //    animation.fillMode = kCAFillModeForwards;
    //    animation.removedOnCompletion = NO;
    //    [videoLayer addAnimation:animation forKey:nil];
    //
    //    {
    //        CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"position.x"];
    //        animation.fromValue = @(0);
    //        animation.toValue = @(360.0f);
    //        animation.beginTime = AVCoreAnimationBeginTimeAtZero;
    //        animation.duration = 0.000f;
    //        animation.fillMode = kCAFillModeForwards;
    //        animation.removedOnCompletion = NO;
    //        [videoLayer addAnimation:animation forKey:nil];
    //    }
    //    {
    //        CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"position.y"];
    //        animation.fromValue = @(0.2f);
    //        animation.toValue = @(640.0f);
    //        animation.beginTime = AVCoreAnimationBeginTimeAtZero;
    //        animation.duration = 0.000f;
    //        animation.fillMode = kCAFillModeForwards;
    //        animation.removedOnCompletion = NO;
    //        [videoLayer addAnimation:animation forKey:nil];
    //    }
    
    
    //俩次放射变换 旋转视频90度，和移动视频
    videoLayer.affineTransform = CGAffineTransformMakeRotation(-3*M_PI/2);
    videoLayer.affineTransform = CGAffineTransformConcat(videoLayer.affineTransform, CGAffineTransformMakeTranslation(-280,280));
    
    
    AVMutableVideoComposition *videoComp = [AVMutableVideoComposition videoComposition];
    videoComp.renderSize = videoSize;
    videoComp.frameDuration = CMTimeMake(1, 30);
    videoComp.animationTool = [AVVideoCompositionCoreAnimationTool videoCompositionCoreAnimationToolWithPostProcessingAsVideoLayer:videoLayer inLayer:parentLayer];
    AVMutableVideoCompositionInstruction *instruction = [AVMutableVideoCompositionInstruction videoCompositionInstruction];
    instruction.timeRange = CMTimeRangeMake(kCMTimeZero, [workbench duration]);
    
    
    AVMutableVideoCompositionLayerInstruction* layerInstruction = [AVMutableVideoCompositionLayerInstruction videoCompositionLayerInstructionWithAssetTrack:videoTrack];
    //改变工作台视频的尺寸为720*1280
    CGFloat changeWidth = 720 / mainVideoTrack.naturalSize.width;
    CGFloat changeHeight = 1280 / mainVideoTrack.naturalSize.height;
    CGAffineTransform chageingScale = CGAffineTransformMakeScale(changeWidth, changeHeight);
    [layerInstruction setTransform:CGAffineTransformConcat(mainVideoTrack.preferredTransform, chageingScale) atTime:kCMTimeZero];
    
    //加入指令
    instruction.layerInstructions = [NSArray arrayWithObject:layerInstruction];
    videoComp.instructions = [NSArray arrayWithObject: instruction];
    
    
    
    AVAssetExportSession *exportSession = [[AVAssetExportSession alloc] initWithAsset:workbench presetName:AVAssetExportPresetHighestQuality];
    exportSession.videoComposition=videoComp;
    exportSession.outputURL = self.url;
    exportSession.outputFileType = AVFileTypeMPEG4;
    [exportSession exportAsynchronouslyWithCompletionHandler:^{
        switch (exportSession.status) {
            case AVAssetExportSessionStatusUnknown:
                NSLog(@"AVAssetExportSessionStatusUnknown");
                break;
            case AVAssetExportSessionStatusWaiting:
                NSLog(@"AVAssetExportSessionStatusWaiting");
                break;
            case AVAssetExportSessionStatusExporting:
                NSLog(@"AVAssetExportSessionStatusExporting");
                break;
            case AVAssetExportSessionStatusFailed:
                NSLog(@"AVAssetExportSessionStatusFailed");
                break;
            case AVAssetExportSessionStatusCancelled:
                NSLog(@"AVAssetExportSessionStatusCancelled");
                break;
            case AVAssetExportSessionStatusCompleted:
                NSLog(@"export succeed");
                dispatch_async(dispatch_get_main_queue(), ^{
                    
                    [self playVideo];
                    [self presentViewController:self.player animated:YES completion:^{
                        [self->hud hideAnimated:true];
                    }];
                });
                break;
        }
    }];
    
    
}
-(void)addGIF
{
    //创建工作台
    AVMutableComposition *workbench = [AVMutableComposition composition];
    //创建主视频和音频轨道添加到工作台
    AVMutableCompositionTrack *mainVideoTrack = [workbench addMutableTrackWithMediaType:AVMediaTypeVideo preferredTrackID:kCMPersistentTrackID_Invalid];
    AVMutableCompositionTrack *mainAudioTrack = [workbench addMutableTrackWithMediaType:AVMediaTypeAudio preferredTrackID:kCMPersistentTrackID_Invalid];
    //引入外部多媒体 创建资源轨道
    AVURLAsset* videoAsset = [[AVURLAsset alloc]initWithURL:[[NSBundle mainBundle] URLForResource:@"v1" withExtension:@"mp4"] options:nil];
    AVAssetTrack *videoTrack = [[videoAsset tracksWithMediaType:AVMediaTypeVideo] objectAtIndex:0];
    AVAssetTrack *audioTrack = [[videoAsset tracksWithMediaType:AVMediaTypeAudio] objectAtIndex:0];
    
    //向音视频轨道插入资源
    [mainVideoTrack insertTimeRange:CMTimeRangeMake(kCMTimeZero, videoAsset.duration) ofTrack:videoTrack atTime:kCMTimeZero error:nil];
    [mainAudioTrack insertTimeRange:CMTimeRangeMake(kCMTimeZero, videoAsset.duration) ofTrack:audioTrack atTime:kCMTimeZero error:nil];
    
    //获取视频资源的尺寸
    CGSize videoSize = [videoTrack naturalSize];
    
    CALayer *imgLay = [CALayer layer];
    imgLay.frame = CGRectMake(100, 100, 200, 200);
    imgLay.backgroundColor = UIColor.redColor.CGColor;
    
    LOTAnimationView* animation = [LOTAnimationView animationNamed:@"青蛙"];
    animation.frame = CGRectMake(150 , 340 , 240 , 240 );
    animation.animationSpeed = 5.0 ;
    animation.loopAnimation = YES;
    [animation play];
    
    
    
    //创建父layer
    CALayer *parentLayer = [CALayer layer];
    parentLayer.frame=CGRectMake(0, 0, videoSize.width, videoSize.height);;
    //准备layer为参数，这个决定视频的大小
    CALayer *videoLayer=[CALayer layer];
    videoLayer.frame=CGRectMake(0, 0, videoSize.width, videoSize.height);
    [parentLayer addSublayer:videoLayer];
    [parentLayer addSublayer:animation.layer];
    //反转坐标
    parentLayer.geometryFlipped = true;
    
    //创建视频工作台
    AVMutableVideoComposition *videoWorkbench=[AVMutableVideoComposition videoComposition];
    //设置每秒播放的帧数
    videoWorkbench.frameDuration=CMTimeMake(1, 30);
    videoWorkbench.renderSize = videoSize;
    //设置视频层为“videoLayer”，设置父视层为“parentLayer”
    videoWorkbench.animationTool=[AVVideoCompositionCoreAnimationTool videoCompositionCoreAnimationToolWithPostProcessingAsVideoLayer:videoLayer inLayer:parentLayer];
    
    
    //创建视频指令
    AVMutableVideoCompositionInstruction *videoinstruction = [AVMutableVideoCompositionInstruction videoCompositionInstruction];
    //设置时间
    videoinstruction.timeRange = CMTimeRangeMake(kCMTimeZero, [workbench duration]);
    //工作台中的媒体资源创建轨道
    AVAssetTrack *workbenchVideoTrack = [[workbench tracksWithMediaType:AVMediaTypeVideo] objectAtIndex:0];
    //创建工作台视频指令层
    AVMutableVideoCompositionLayerInstruction* layerInstruction = [AVMutableVideoCompositionLayerInstruction videoCompositionLayerInstructionWithAssetTrack:workbenchVideoTrack];
    //视频指令中加入主工作台的资源
    videoinstruction.layerInstructions = [NSArray arrayWithObject:layerInstruction];
    //添加主工作台到“videoWorkbench”工作台中
    videoWorkbench.instructions = [NSArray arrayWithObject: videoinstruction];
    
    
    
    AVAssetExportSession *exportSession = [[AVAssetExportSession alloc] initWithAsset:workbench presetName:AVAssetExportPresetMediumQuality];
    exportSession.videoComposition=videoWorkbench;
    
    exportSession.outputURL = self.url;
    exportSession.outputFileType = AVFileTypeQuickTimeMovie;
    [exportSession exportAsynchronouslyWithCompletionHandler:^{
        switch (exportSession.status) {
            case AVAssetExportSessionStatusUnknown:
                NSLog(@"AVAssetExportSessionStatusUnknown");
                break;
            case AVAssetExportSessionStatusWaiting:
                NSLog(@"AVAssetExportSessionStatusWaiting");
                break;
            case AVAssetExportSessionStatusExporting:
                NSLog(@"AVAssetExportSessionStatusExporting");
                break;
            case AVAssetExportSessionStatusFailed:
                NSLog(@"AVAssetExportSessionStatusFailed");
                break;
            case AVAssetExportSessionStatusCancelled:
                NSLog(@"AVAssetExportSessionStatusCancelled");
                break;
            case AVAssetExportSessionStatusCompleted:
                NSLog(@"export succeed");
                dispatch_async(dispatch_get_main_queue(), ^{
                    
                    [self playVideo];
                    [self presentViewController:self.player animated:YES completion:^{
                        [self->hud hideAnimated:true];
                    }];
                });
                break;
        }
    }];
}
-(void)addPngAnimation
{
    //创建工作台
    AVMutableComposition *workbench = [AVMutableComposition composition];
    //创建主视频和音频轨道添加到工作台
    AVMutableCompositionTrack *mainVideoTrack = [workbench addMutableTrackWithMediaType:AVMediaTypeVideo preferredTrackID:kCMPersistentTrackID_Invalid];
    AVMutableCompositionTrack *mainAudioTrack = [workbench addMutableTrackWithMediaType:AVMediaTypeAudio preferredTrackID:kCMPersistentTrackID_Invalid];
    //引入外部多媒体 创建资源轨道
    AVURLAsset* videoAsset = [[AVURLAsset alloc]initWithURL:[[NSBundle mainBundle] URLForResource:@"v1" withExtension:@"mp4"] options:nil];
    AVAssetTrack *videoTrack = [[videoAsset tracksWithMediaType:AVMediaTypeVideo] objectAtIndex:0];
    AVAssetTrack *audioTrack = [[videoAsset tracksWithMediaType:AVMediaTypeAudio] objectAtIndex:0];
    
    //向音视频轨道插入资源
    [mainVideoTrack insertTimeRange:CMTimeRangeMake(kCMTimeZero, videoAsset.duration) ofTrack:videoTrack atTime:kCMTimeZero error:nil];
    [mainAudioTrack insertTimeRange:CMTimeRangeMake(kCMTimeZero, videoAsset.duration) ofTrack:audioTrack atTime:kCMTimeZero error:nil];
    
    //获取视频资源的尺寸
    CGSize videoSize = [videoTrack naturalSize];
    
    CALayer *gifLayer1 = [[CALayer alloc] init];
    gifLayer1.frame = CGRectMake(150 , 340 , 298 , 253 );
    CAKeyframeAnimation *gifLayer1Animation = [self animationForGifWithURL:[[NSBundle mainBundle] URLForResource:@"son" withExtension:@"gif"]];
    gifLayer1Animation.beginTime = AVCoreAnimationBeginTimeAtZero;
    gifLayer1Animation.removedOnCompletion = NO;
    [gifLayer1 addAnimation:gifLayer1Animation forKey:@"gif"];
    
    
    
    //创建父layer
    CALayer *parentLayer = [CALayer layer];
    parentLayer.frame=CGRectMake(0, 0, videoSize.width, videoSize.height);;
    //准备layer为参数，这个决定视频的大小
    CALayer *videoLayer=[CALayer layer];
    videoLayer.frame=CGRectMake(0, 0, videoSize.width, videoSize.height);
    [parentLayer addSublayer:videoLayer];
    [parentLayer addSublayer:gifLayer1];
    //反转坐标
    parentLayer.geometryFlipped = true;
    
    //创建视频工作台
    AVMutableVideoComposition *videoWorkbench=[AVMutableVideoComposition videoComposition];
    //设置每秒播放的帧数
    videoWorkbench.frameDuration=CMTimeMake(1, 30);
    videoWorkbench.renderSize = videoSize;
    //设置视频层为“videoLayer”，设置父视层为“parentLayer”
    videoWorkbench.animationTool=[AVVideoCompositionCoreAnimationTool videoCompositionCoreAnimationToolWithPostProcessingAsVideoLayer:videoLayer inLayer:parentLayer];
    
    
    //创建视频指令
    AVMutableVideoCompositionInstruction *videoinstruction = [AVMutableVideoCompositionInstruction videoCompositionInstruction];
    //设置时间
    videoinstruction.timeRange = CMTimeRangeMake(kCMTimeZero, [workbench duration]);
    //工作台中的媒体资源创建轨道
    AVAssetTrack *workbenchVideoTrack = [[workbench tracksWithMediaType:AVMediaTypeVideo] objectAtIndex:0];
    //创建工作台视频指令层
    AVMutableVideoCompositionLayerInstruction* layerInstruction = [AVMutableVideoCompositionLayerInstruction videoCompositionLayerInstructionWithAssetTrack:workbenchVideoTrack];
    //视频指令中加入主工作台的资源
    videoinstruction.layerInstructions = [NSArray arrayWithObject:layerInstruction];
    //添加主工作台到“videoWorkbench”工作台中
    videoWorkbench.instructions = [NSArray arrayWithObject: videoinstruction];
    
    
    
    AVAssetExportSession *exportSession = [[AVAssetExportSession alloc] initWithAsset:workbench presetName:AVAssetExportPresetMediumQuality];
    exportSession.videoComposition=videoWorkbench;
    
    exportSession.outputURL = self.url;
    exportSession.outputFileType = AVFileTypeQuickTimeMovie;
    [exportSession exportAsynchronouslyWithCompletionHandler:^{
        switch (exportSession.status) {
            case AVAssetExportSessionStatusUnknown:
                NSLog(@"AVAssetExportSessionStatusUnknown");
                break;
            case AVAssetExportSessionStatusWaiting:
                NSLog(@"AVAssetExportSessionStatusWaiting");
                break;
            case AVAssetExportSessionStatusExporting:
                NSLog(@"AVAssetExportSessionStatusExporting");
                break;
            case AVAssetExportSessionStatusFailed:
                NSLog(@"AVAssetExportSessionStatusFailed");
                break;
            case AVAssetExportSessionStatusCancelled:
                NSLog(@"AVAssetExportSessionStatusCancelled");
                break;
            case AVAssetExportSessionStatusCompleted:
                NSLog(@"export succeed");
                dispatch_async(dispatch_get_main_queue(), ^{
                    
                    [self playVideo];
                    [self presentViewController:self.player animated:YES completion:^{
                        [self->hud hideAnimated:true];
                    }];
                });
                break;
        }
    }];
}
-(CAKeyframeAnimation *)animationForGifWithURL:(NSURL *)url {
    
    CAKeyframeAnimation *animation = [CAKeyframeAnimation animationWithKeyPath:@"contents"];
    
    NSMutableArray * frames = [NSMutableArray new];
    NSMutableArray *delayTimes = [NSMutableArray new];
    
    CGFloat totalTime = 0.0;
    CGFloat gifWidth;
    CGFloat gifHeight;
    
    CGImageSourceRef gifSource = CGImageSourceCreateWithURL((CFURLRef)url, NULL);
    
    // get frame count
    size_t frameCount = CGImageSourceGetCount(gifSource);
    for (size_t i = 0; i < frameCount; ++i) {
        // get each frame
        CGImageRef frame = CGImageSourceCreateImageAtIndex(gifSource, i, NULL);
        [frames addObject:(__bridge id)frame];
        CGImageRelease(frame);
        
        // get gif info with each frame
        NSDictionary *dict = (NSDictionary*)CFBridgingRelease(CGImageSourceCopyPropertiesAtIndex(gifSource, i, NULL));
        NSLog(@"kCGImagePropertyGIFDictionary %@", [dict valueForKey:(NSString*)kCGImagePropertyGIFDictionary]);
        
        // get gif size
        gifWidth = [[dict valueForKey:(NSString*)kCGImagePropertyPixelWidth] floatValue];
        gifHeight = [[dict valueForKey:(NSString*)kCGImagePropertyPixelHeight] floatValue];
        
        // kCGImagePropertyGIFDictionary中kCGImagePropertyGIFDelayTime，kCGImagePropertyGIFUnclampedDelayTime值是一样的
        NSDictionary *gifDict = [dict valueForKey:(NSString*)kCGImagePropertyGIFDictionary];
        [delayTimes addObject:[gifDict valueForKey:(NSString*)kCGImagePropertyGIFUnclampedDelayTime]];
        
        totalTime = totalTime + [[gifDict valueForKey:(NSString*)kCGImagePropertyGIFUnclampedDelayTime] floatValue];
        
        //        CFRelease((__bridge CFTypeRef)(dict));
        //        CFRelease((__bridge CFTypeRef)(dict));
    }
    if (gifSource) {
        CFRelease(gifSource);
    }
    
    NSMutableArray *times = [NSMutableArray arrayWithCapacity:3];
    CGFloat currentTime = 0;
    NSInteger count = delayTimes.count;
    for (int i = 0; i < count; ++i) {
        [times addObject:[NSNumber numberWithFloat:(currentTime / totalTime)]];
        currentTime += [[delayTimes objectAtIndex:i] floatValue];
    }
    
    NSMutableArray *images = [NSMutableArray arrayWithCapacity:3];
    for (int i = 0; i < count; ++i) {
        [images addObject:[frames objectAtIndex:i]];
    }
    
    animation.keyTimes = times;
    animation.values = images;
    animation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionLinear];
    animation.duration = totalTime;
    animation.repeatCount = HUGE_VALF;
    
    return animation;
}
@end

</pre>
