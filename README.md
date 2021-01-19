# IJKPlayer-Build-Steps

Preface
The rtmp stream is pushed on Mac, and ijkPlayer is integrated for iOS playback.

step
1. Let's run ijkPlayerDemo first

Follow the Before Build in the README .

git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-ios
cd ijkplayer-ios
git checkout -B latest k0.8.8

./init-ios.sh

Compile openssl to support https, does not need to be skipped
 ./init-ios-openssl.sh 
echo 'Export COMMON_FF_CFG_FLAGS = "$ COMMON_FF_CFG_FLAGS --enable-openssl"' >> ../config/module.sh 
./compile- openssl.sh all

cd ios
./compile-ffmpeg.sh clean
./compile-ffmpeg.sh all

In ./compile-ffmpeg.sh allthe time will complain
./libavutil/arm/asm.S:50:9: error: unknown directive
.arch armv7-a
-----------------solution------------------
vim ios/tools/do-compile-ffmpeg.sh

add FFMPEG_CFG_FLAGS_ARM="$FFMPEG_CFG_FLAGS_ARM --disable-asm"

 # armv7, armv7s, arm64
 FFMPEG_CFG_FLAGS_ARM=
 FFMPEG_CFG_FLAGS_ARM="$FFMPEG_CFG_FLAGS_ARM --enable-pic"
 FFMPEG_CFG_FLAGS_ARM="$FFMPEG_CFG_FLAGS_ARM --enable-neon"
 FFMPEG_CFG_FLAGS_ARM="$FFMPEG_CFG_FLAGS_ARM --disable-asm"
-------------------------------------

2. Package the static library IJKMediaFramework

Open IJKMediaPlayer.xcodeproj , select IJKMediaFramework——Run——Info——Build Configuration——Release as the scheme , select the simulator build, and rebuild on the real machine will report an error

'armv7/avconfig.h' file not found
The build succeeded after commenting out this line.

Project Directory Products under the right IJKMediaFramework.framework - Show in the Finder , in Release-iphoneos under the path where the merger and the real machine simulator common static library

lipo -create Release-iphoneos/IJKMediaFramework.framework/IJKMediaFramework Release-iphonesimulator/IJKMediaFramework.framework/IJKMediaFramework -output IJKMediaFramework
Replace the generated IJKMediaFramework with the path Release-iphoneos/IJKMediaFramework.framework/IJKMediaFramework

3. New works added after IJKMediaFramework.framework and system dependencies, then also add support https libcrypto.aand libssl.a . Run error

"operator delete(void*)", referenced from:
Add system dependency library libc++.tbd to solve

4. Use ijkPlayer

var player: IJKFFMoviePlayerController!

override func viewDidLoad() {
        super.viewDidLoad()
        
        player = IJKFFMoviePlayerController.init(contentURL: URL.init(string: "rtmp://192.168.69.21:1990/liveApp")!, with: IJKFFOptions.byDefault())
        player.view.frame = CGRect(x: 0, y: 0, width: 300, height: 300)
        view.addSubview(player.view)
        player.prepareToPlay()
        player.play()
        // Do any additional setup after loading the view, typically from a nib.
    }
Live streaming on Mac
