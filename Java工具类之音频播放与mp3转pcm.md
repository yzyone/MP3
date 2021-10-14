
### Java工具类之音频播放与mp3转pcm ###

小程序通过科大讯飞或百度语音识别，由于无法识别mp3文件，需要将mp3文件转换为pcm文件（mp3的音频包含文件头描述啥的，而pcm的音频格式就纯音频了，没有文件头信息）。

依赖jar包

	<dependency>
	    <groupId>com.googlecode.soundlibs</groupId>
	    <artifactId>mp3spi</artifactId>
	    <version>1.9.5.4</version>
	</dependency>

工具代码

```
package com.yellowcong.baidu.utils;
 
import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.IOException;
import java.io.OutputStream;
 
import javax.sound.sampled.AudioFileFormat;
import javax.sound.sampled.AudioFormat;
import javax.sound.sampled.AudioInputStream;
import javax.sound.sampled.AudioSystem;
import javax.sound.sampled.DataLine;
import javax.sound.sampled.SourceDataLine;
 
import javazoom.spi.mpeg.sampled.file.MpegAudioFileReader;
 
/**
 * 创建日期:2018年1月14日
 * 创建时间:下午10:09:39
 * 创建者    :yellowcong
 * 机能概要:MP3转PCM Java方式实现
 * http://ai.baidu.com/forum/topic/show/496972
 */
public class AudioUtils {
    private static AudioUtils audioUtils = null;
    private AudioUtils(){}
 
    //双判断，解决单利问题
    public static AudioUtils getInstance(){
        if(audioUtils == null){
            synchronized (AudioUtils.class) {
                if(audioUtils == null){
                    audioUtils = new AudioUtils();
                }
            }
        }
        return audioUtils;
    }
 
    /**
     * MP3转换PCM文件方法
     * 
     * @param mp3filepath 原始文件路径
     * @param pcmfilepath 转换文件的保存路径
     * @return 
     * @throws Exception
     */
    public boolean convertMP32Pcm(String mp3filepath, String pcmfilepath){
        try {
            //获取文件的音频流，pcm的格式
            AudioInputStream audioInputStream = getPcmAudioInputStream(mp3filepath);
            //将音频转化为  pcm的格式保存下来
            AudioSystem.write(audioInputStream, AudioFileFormat.Type.WAVE, new File(pcmfilepath));
            return true;
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
            return false;
        }
    }
 
    /**
     * 播放MP3方法
     * 
     * @param mp3filepath
     * @throws Exception
     */
    public void playMP3(String mp3filepath) throws Exception {
        //获取音频为pcm的格式
        AudioInputStream audioInputStream = getPcmAudioInputStream(mp3filepath);
 
        // 播放
        if (audioInputStream == null){
            System.out.println("null audiostream");
            return;
        }
        //获取音频的格式
        AudioFormat targetFormat = audioInputStream.getFormat();
        DataLine.Info dinfo = new DataLine.Info(SourceDataLine.class, targetFormat, AudioSystem.NOT_SPECIFIED);
        //输出设备
        SourceDataLine line = null;
        try {
            line = (SourceDataLine) AudioSystem.getLine(dinfo);
            line.open(targetFormat);
            line.start();
 
            int len = -1;
//            byte[] buffer = new byte[8192];
            byte[] buffer = new byte[1024];
            //读取音频文件
            while ((len = audioInputStream.read(buffer)) > 0) {
                //输出音频文件
                line.write(buffer, 0, len);
            }
 
            // Block等待临时数据被输出为空
            line.drain();
 
            //关闭读取流
            audioInputStream.close();
 
            //停止播放
            line.stop();
            line.close();
 
        } catch (Exception ex) {
            ex.printStackTrace();
            System.out.println("audio problem " + ex);
 
        }
    }
 
    /**
     * 创建日期:2018年1月14日<br/>
     * 创建时间:下午9:53:14<br/>
     * 创建用户:yellowcong<br/>
     * 机能概要:获取文件的音频流
     * @param mp3filepath
     * @return
     */
    private AudioInputStream getPcmAudioInputStream(String mp3filepath) {
        File mp3 = new File(mp3filepath);
        AudioInputStream audioInputStream = null;
        AudioFormat targetFormat = null;
        try {
            AudioInputStream in = null;
 
            //读取音频文件的类
            MpegAudioFileReader mp = new MpegAudioFileReader();
            in = mp.getAudioInputStream(mp3);
            AudioFormat baseFormat = in.getFormat();
 
            //设定输出格式为pcm格式的音频文件
            targetFormat = new AudioFormat(AudioFormat.Encoding.PCM_SIGNED, baseFormat.getSampleRate(), 16,
                    baseFormat.getChannels(), baseFormat.getChannels() * 2, baseFormat.getSampleRate(), false);
 
            //输出到音频
            audioInputStream = AudioSystem.getAudioInputStream(targetFormat, in);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return audioInputStream;
    }
}
```

测试代码

```
package yellowcong.yuyin;
 
import org.junit.Test;
 
import com.yellowcong.baidu.utils.AudioUtils;
 
/**
 * 创建日期:2018年1月14日
 * 创建时间:下午10:09:39
 * 创建者    :yellowcong
 * 机能概要:MP3转PCM Java方式实现
 */
public class TestAudioUtils {
    //测试播放音频
    @Test
    public void testPaly() throws Exception{
        AudioUtils utils  = AudioUtils.getInstance();
        utils.playMP3("D:/xx.mp3");
    }
 
    @Test
    public void testConvert() throws Exception{
        AudioUtils utils  = AudioUtils.getInstance();
        utils.convertMP32Pcm("D:/xx.mp3", "D:/xx.pcm");
    }
}
```

原文链接：https://blog.csdn.net/yelllowcong/article/details/79059847